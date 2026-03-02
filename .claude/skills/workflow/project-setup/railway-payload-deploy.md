# Railway Deployment - Payload CMS Backend

Step-by-step guide for deploying a Payload CMS backend on Railway with PostgreSQL. This covers the initial setup when a content website project picks Railway as the backend platform.

## Prerequisites

- Railway CLI installed (`brew install railway` or `curl -fsSL https://railway.com/install.sh | sh`)
- Logged in (`railway login`)
- GitHub repo connected to Railway (done once in Railway dashboard under Settings > Source)
- Project code pushed to GitHub with the monorepo structure:
  ```
  repo/
  ├── backend/          # Payload CMS (has its own package.json + Dockerfile)
  └── frontend/         # Astro (deployed separately)
  ```

## Step 1: Create Railway Project

```bash
railway init --name <project-name>
```

This creates and links the project in one step. Verify with `railway status --json`.

## Step 2: Add Postgres Database

```bash
railway add --database postgres
```

This provisions a managed Postgres 17 instance with a persistent volume. Railway auto-creates connection variables (`DATABASE_URL`, `PGHOST`, `PGPORT`, etc.) on the Postgres service.

Verify the database is running:

```bash
railway status --json
```

Look for a service with `source.image` containing `postgres`.

## Step 3: Create the Backend Service

Create an empty service for the Payload CMS backend:

```bash
railway add --service <project-name>
```

Then connect it to the GitHub repo:

```bash
railway environment edit --service-config <project-name> source.repo <github-user>/<repo-name>
railway environment edit --service-config <project-name> source.branch main
```

## Step 4: Configure Monorepo Paths

Set the root directory to `/backend` so Railway only builds from the backend folder:

```bash
railway environment edit --service-config <project-name> source.rootDirectory "/backend"
```

If the backend uses a Dockerfile (recommended for Payload), configure the builder:

```bash
railway environment edit --service-config <project-name> build.builder DOCKERFILE
railway environment edit --service-config <project-name> build.dockerfilePath "backend/Dockerfile"
```

If not using a Dockerfile, Railway's Railpack builder auto-detects Node.js from `package.json`.

## Step 5: Set Environment Variables

Payload CMS requires exactly 3 environment variables on Railway:

```bash
# 1. Database connection - wires to the Postgres service via Railway template syntax
railway variable set 'DATABASE_URI=${{Postgres.DATABASE_URL}}' --service <project-name>

# 2. Public URL - auto-resolves to the service's Railway domain
railway variable set 'PAYLOAD_PUBLIC_SERVER_URL=https://${{RAILWAY_PUBLIC_DOMAIN}}' --service <project-name>

# 3. Secret key - generate a fresh one for each project
railway variable set "PAYLOAD_SECRET=$(openssl rand -hex 32)" --service <project-name>
```

### Variable Details

| Variable | Value | Purpose |
|----------|-------|---------|
| `DATABASE_URI` | `${{Postgres.DATABASE_URL}}` | Wires Payload to the Railway Postgres service. Uses Railway's template syntax to auto-resolve the connection string. |
| `PAYLOAD_PUBLIC_SERVER_URL` | `https://${{RAILWAY_PUBLIC_DOMAIN}}` | The public URL where Payload is accessible. Uses Railway's auto-provided domain variable so it stays in sync. |
| `PAYLOAD_SECRET` | `openssl rand -hex 32` | Random 64-character hex string for Payload's auth and encryption. Must be unique per project - never reuse across projects. |

**Important:** `DATABASE_URI` (not `DATABASE_URL`) - this is the variable name Payload CMS expects. The Postgres service provides `DATABASE_URL`, but the Payload service must map it to `DATABASE_URI`.

## Step 6: Generate Railway Domain

The `PAYLOAD_PUBLIC_SERVER_URL` uses `${{RAILWAY_PUBLIC_DOMAIN}}`, which only resolves after a domain is assigned:

```bash
railway domain --service <project-name>
```

This generates a `<project-name>-production.up.railway.app` domain. Optionally add a custom domain:

```bash
railway domain cms.example.com --service <project-name>
```

## Step 7: Deploy

If the GitHub repo is connected and source is configured, Railway auto-deploys on push to `main`. For a manual first deploy:

```bash
railway up --service <project-name> --detach -m "Initial Payload CMS deployment"
```

## Step 8: Verify

```bash
# Check deployment status
railway status --json

# Check build logs if deployment fails
railway logs --service <project-name> --build --lines 100

# Check runtime logs
railway logs --service <project-name> --lines 50
```

Once deployed, visit the Railway domain + `/admin` to create the first admin account.

## Frontend Deployment

The Astro frontend deploys separately - not on Railway. Common choices:

| Platform | How |
|----------|-----|
| Cloudflare Pages | Connect GitHub repo, set root directory to `frontend/`, build command `bun run build` |
| Netlify | Connect GitHub repo, set base directory to `frontend/`, build command `bun run build` |
| Vercel | Connect GitHub repo, set root directory to `frontend/`, framework preset "Astro" |

The frontend needs `PAYLOAD_URL` pointing to the Railway Payload domain (the same URL from step 6) to fetch content at build time.

## Quick Reference

Full setup in order:

```bash
# 1. Create project
railway init --name my-website

# 2. Add Postgres
railway add --database postgres

# 3. Add backend service
railway add --service my-website
railway environment edit --service-config my-website source.repo user/repo
railway environment edit --service-config my-website source.branch main
railway environment edit --service-config my-website source.rootDirectory "/backend"
railway environment edit --service-config my-website build.builder DOCKERFILE
railway environment edit --service-config my-website build.dockerfilePath "backend/Dockerfile"

# 4. Set variables
railway variable set 'DATABASE_URI=${{Postgres.DATABASE_URL}}' --service my-website
railway variable set 'PAYLOAD_PUBLIC_SERVER_URL=https://${{RAILWAY_PUBLIC_DOMAIN}}' --service my-website
railway variable set "PAYLOAD_SECRET=$(openssl rand -hex 32)" --service my-website

# 5. Generate domain
railway domain --service my-website

# 6. Deploy (or push to main for auto-deploy)
railway up --service my-website --detach -m "Initial deploy"
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Build fails with Dockerfile not found | Verify `dockerfilePath` matches the actual path (e.g., `backend/Dockerfile`) |
| `DATABASE_URI` connection refused | Check Postgres service is running, verify variable uses `${{Postgres.DATABASE_URL}}` exactly |
| `PAYLOAD_PUBLIC_SERVER_URL` is empty | Generate a domain first (`railway domain`), then redeploy |
| Admin panel 404 | Check start command runs Payload on the correct port, Railway expects `PORT` env var |
| Payload exceeds memory | Upgrade from hobby to pro plan, or optimize Payload config |
