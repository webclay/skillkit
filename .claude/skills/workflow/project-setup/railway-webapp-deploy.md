# Railway Deployment - Web App (Production + Staging)

Step-by-step guide for deploying a web app on Railway with two environments - Production and Staging - each auto-deploying from a different GitHub branch.

## Architecture

Two Railway environments inside a single Railway project:

| Environment | Branch | Domain | Purpose |
|-------------|--------|--------|---------|
| Production | `main` | yourdomain.com | Live users |
| Staging | `staging` | staging.yourdomain.com | Testing before production |

Each environment runs identical services (app, database, redis, etc.) with separate configs, secrets, and domains.

## Git Branching Strategy

Three branch tiers:

| Branch | Purpose | Who pushes |
|--------|---------|-----------|
| `feature/*` | Individual task work | Claude / developer |
| `staging` | Integration branch, code review | PRs from feature branches |
| `main` | Production-ready code | PRs from staging only |

Flow:
```
feature/task-name → PR → staging → PR → main → Railway auto-deploys
```

**Rules:**
- Never push directly to `staging` or `main`
- Always use feature branches with PRs
- Feature PRs target `staging`, not `main`
- Only merge `staging` → `main` when ready to go live

## Prerequisites

- Railway CLI installed (`brew install railway` or `curl -fsSL https://railway.com/install.sh | sh`)
- Logged in (`railway login`)
- GitHub repo with `main` and `staging` branches created

## Step 1: Create the Staging Branch

```bash
git checkout -b staging
git push -u origin staging
```

## Step 2: Create Railway Project

```bash
railway init --name <project-name>
```

Verify with `railway status`.

## Step 3: Add Services to Production Environment

Railway starts in the `production` environment by default. Add services here first.

```bash
# App service
railway add --service <project-name>

# Database
railway add --database postgres

# Redis (if needed)
railway add --database redis
```

Connect the app service to `main`:

```bash
railway environment edit --service-config <project-name> source.repo <github-user>/<repo-name>
railway environment edit --service-config <project-name> source.branch main
```

Set start command (example for TanStack Start / Node):

```bash
railway environment edit --service-config <project-name> deploy.startCommand "node .output/server/index.mjs"
```

## Step 4: Set Production Environment Variables

```bash
railway variable set DATABASE_URL='${{Postgres.DATABASE_URL}}' --service <project-name>
railway variable set REDIS_URL='${{Redis.REDIS_URL}}' --service <project-name>  # if using Redis
railway variable set NODE_ENV=production --service <project-name>
# Add all other app-specific vars
```

## Step 5: Generate Production Domain

```bash
railway domain --service <project-name>
```

This creates `<project-name>-production.up.railway.app`. Add custom domains:

```bash
railway domain yourdomain.com --service <project-name>
railway domain www.yourdomain.com --service <project-name>
```

## Step 6: Create Staging Environment

In the Railway dashboard: **Project Settings > Environments > New Environment**, name it `Staging`.

Switch to it:

```bash
railway environment staging
```

## Step 7: Add Services to Staging Environment

Add the same services in the staging environment:

```bash
railway add --service <project-name>
railway add --database postgres
railway add --database redis  # if needed
```

Connect the staging app service to `staging` branch:

```bash
railway environment edit --service-config <project-name> source.repo <github-user>/<repo-name>
railway environment edit --service-config <project-name> source.branch staging
railway environment edit --service-config <project-name> deploy.startCommand "node .output/server/index.mjs"
```

## Step 8: Set Staging Environment Variables

```bash
railway variable set DATABASE_URL='${{Postgres.DATABASE_URL}}' --service <project-name>
railway variable set REDIS_URL='${{Redis.REDIS_URL}}' --service <project-name>  # if needed
railway variable set NODE_ENV=staging --service <project-name>
# Add all other app-specific vars (same as production, different values where needed)
```

## Step 9: Generate Staging Domain

```bash
railway domain --service <project-name>
railway domain staging.yourdomain.com --service <project-name>
```

## Step 10: Auth and CORS Config

Make sure your auth config trusts both domains:

```
yourdomain.com
www.yourdomain.com
staging.yourdomain.com
*.railway.app  # for testing before custom domains are set up
```

If you have a separate backend, whitelist both production and staging origins in your CORS config.

## Step 11: Database Sync (Recommended)

Keep staging in sync with production data by adding a GitHub Action that dumps production and restores into staging on every push to `main`.

Create `.github/workflows/sync-db-to-staging.yml`:

```yaml
name: Sync Production DB to Staging

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Install postgres client
        run: sudo apt-get install -y postgresql-client

      - name: Sync production DB to staging
        run: |
          pg_dump $PRODUCTION_DATABASE_URL | psql $STAGING_DATABASE_URL
        env:
          PRODUCTION_DATABASE_URL: ${{ secrets.PRODUCTION_DATABASE_URL }}
          STAGING_DATABASE_URL: ${{ secrets.STAGING_DATABASE_URL }}
```

Add both secrets to the GitHub repo (Settings > Secrets and variables > Actions):
- `PRODUCTION_DATABASE_URL` - the Railway Postgres URL from production environment
- `STAGING_DATABASE_URL` - the Railway Postgres URL from staging environment

**Effect:** Staging gets a fresh copy of production data on every production deploy. Staging and production share the same data shape; staging is a snapshot.

## Step 12: Update Wrap-up Workflow

The `/wrap-up` skill creates PRs. For this project, feature branches should PR into `staging`, not `main`. Document this in `project/dev-context.md`:

```markdown
## Git Workflow

Branching: feature/* → staging → main
PR target for feature branches: staging
PR target for staging: main
```

The wrap-up skill reads dev-context.md and will use this as the PR target.

## Verify

```bash
# Check production
railway environment production
railway status --json

# Check staging
railway environment staging
railway status --json
```

Push a commit to `staging` to verify Railway auto-deploys. Push to `main` to verify production auto-deploys and (if configured) the DB sync action runs.

## Summary Checklist

- [ ] `staging` branch created and pushed to GitHub
- [ ] Railway project created
- [ ] Production environment: app + postgres + redis services configured, connected to `main`
- [ ] Production environment variables set
- [ ] Production domain generated + custom domain added
- [ ] Staging environment created in Railway
- [ ] Staging environment: same services configured, connected to `staging`
- [ ] Staging environment variables set
- [ ] Staging domain generated + staging.yourdomain.com added
- [ ] Auth trusted origins include both domains
- [ ] CORS config includes both domains (if applicable)
- [ ] DB sync GitHub Action added (optional but recommended)
- [ ] `project/dev-context.md` updated with git workflow (PR target: staging)

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Staging not auto-deploying | Verify `source.branch` is set to `staging` in the Railway staging environment, not production |
| Variables missing in staging | Railway environments are fully isolated - set all variables separately in each environment |
| Custom domain not resolving | Add CNAME or A record pointing to the Railway domain. Allow 5-10 min for DNS propagation |
| DB sync action fails | Verify `PRODUCTION_DATABASE_URL` and `STAGING_DATABASE_URL` are set in GitHub repo secrets |
| Auth errors on staging domain | Add `staging.yourdomain.com` and `*.railway.app` to trusted origins in auth config |
