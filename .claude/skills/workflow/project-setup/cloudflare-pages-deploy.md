# Cloudflare Pages Deployment - Astro Frontend

Step-by-step guide for deploying the Astro frontend to Cloudflare Pages. This covers the full setup when a content website project picks Cloudflare Pages as the frontend platform.

## Prerequisites

- A GitHub repository with the monorepo structure:
  ```
  repo/
  ├── backend/          # Payload CMS (deployed separately - Railway, Cloudflare Workers, etc.)
  └── frontend/         # Astro (deployed to Cloudflare Pages)
  ```
- A Cloudflare account (free tier is sufficient for Pages)
- Backend (Payload CMS) already deployed and accessible via a public URL

## Step 1: Install Dependencies

```bash
# Frontend (Astro) - uses bun
cd frontend && bun install

# Backend (Payload) - uses bun for installing, but pnpm for running
cd backend && bun install
```

### Dev Server Commands

The frontend and backend use different runtimes for development:

| Server | Command | Why |
|--------|---------|-----|
| Frontend (Astro) | `bun run dev` | Full Bun support, runs on localhost:4321 |
| Backend (Payload) | `pnpm run dev` | Payload requires Node.js runtime, runs on localhost:3000 |

**Important:** Payload does not support Bun as a runtime (only as a package manager). The backend keeps pnpm for running dev server and CLI commands like migrations and type generation. See the Payload deployment skill for full details on Bun/Node.js compatibility.

## Step 2: First Commit & Push to GitHub

Cloudflare Pages needs at least one branch pushed to connect a repository.

```bash
git add .
git commit -m "Initial commit"
git branch -M main
git push -u origin main
```

If the repo already has commits pushed, skip this step.

## Step 3: Deploy Backend First

The Payload backend must be live before connecting Cloudflare Pages, because the frontend needs `PUBLIC_PAYLOAD_URL` as an environment variable to fetch content at build time.

**Deploy the backend based on your choice:**

| Platform | Guide |
|----------|-------|
| Railway | See [railway-payload-deploy.md](railway-payload-deploy.md) |
| Other | Deploy Payload to your chosen platform and note the public URL |

After deployment, note the backend URL (e.g., `https://my-project-production.up.railway.app`).

## Step 4: Set Up Database

Database setup depends on your backend choice. If you followed the Railway guide, Postgres is already provisioned.

Make sure the backend is running and accessible before proceeding.

## Step 5: Connect Cloudflare Pages to GitHub

1. Go to the [Cloudflare dashboard](https://dash.cloudflare.com)
2. Navigate to **Workers & Pages**
3. Click **Create** (or **Create application**)
4. Select the **Pages** tab
5. Click **Connect to Git**
6. Select your GitHub repository (e.g., `grasman79/my-project`)
7. Click **Begin setup**

## Step 6: Configure Build Settings

On the "Set up builds and deployments" screen, enter these exact values:

| Setting | Value |
|---------|-------|
| Project name | Your project name (auto-filled from repo, can customize) |
| Production branch | `main` |
| Framework preset | `None` (leave as None - we set the build command manually) |
| Build command | `bun install && bun run build` |
| Build output directory | *Leave default* (controlled by `wrangler.jsonc`) |
| Root directory (advanced) | `frontend` |

**Important:** The build output directory is controlled by `pages_build_output_dir` in `frontend/wrangler.jsonc` (set to `./dist/client`). The Cloudflare dashboard locks this field when a wrangler config exists. Do NOT try to change it in the dashboard - modify `wrangler.jsonc` instead.

### Environment Variables

Click **+ Add variable** and add:

| Type | Name | Value |
|------|------|-------|
| Plaintext | `PUBLIC_PAYLOAD_URL` | Your deployed Payload backend URL (e.g., `https://my-project-production.up.railway.app`) |

**Note:** The build command includes `bun install` because Cloudflare Pages needs to install the frontend dependencies within the `frontend/` root directory before building.

## Step 7: Save and Deploy

Click **Save and Deploy**. The first build starts immediately.

- Cloudflare Pages installs dependencies, runs the build, and deploys to its global CDN
- Your site will be available at `{project-name}.pages.dev`
- Automatic deployments are enabled by default - every push to `main` triggers a new build
- Pull requests get preview deployments automatically

## Step 8: Set Up Deploy Hook (Required)

The deploy-tracker plugin in Payload needs a Cloudflare deploy hook to trigger rebuilds when content changes.

1. In your Pages project, go to **Settings > Build > Deploy hooks**
2. Click **Add deploy hook**
3. Name it `Payload` and select the `main` branch
4. Copy the generated webhook URL
5. In Railway (or wherever Payload is hosted), add the environment variable:
   ```
   CLOUDFLARE_DEPLOY_HOOK_URL=https://api.cloudflare.com/client/v4/pages/webhooks/deploy_hooks/{your-hook-id}
   ```
6. Redeploy the backend so it picks up the new env var

Now editors can click "Deploy Website" in the Payload admin panel to trigger a Cloudflare Pages rebuild.

## Step 9: Verify Post-Deploy Settings

After the first deploy completes, go to the Pages project **Settings** tab and verify:

### Build Settings

| Setting | Expected Value |
|---------|---------------|
| Git repository | Your GitHub repo |
| Build command | `bun install && bun run build` |
| Build output | `dist/client` (locked - controlled by wrangler.jsonc) |
| Root directory | `frontend` |
| Build system version | Version 3 |
| Branch control | Production: main, Automatic deployments: Enabled |

### Runtime Settings

| Setting | Value |
|---------|-------|
| Compatibility flags | `nodejs_compat` |
| Compatibility date | Recent date (e.g., Apr 1, 2025) |

### Variables and Secrets

| Type | Name | Value |
|------|------|-------|
| Plaintext | `PUBLIC_PAYLOAD_URL` | Your Payload backend URL |

## Step 10: Configure Staging Environment

Cloudflare Pages automatically deploys every branch. The `staging` branch gets a preview URL at `staging.{project}.pages.dev`.

### Set up separate environment variables for staging

1. In your Pages project, go to **Settings > Environment variables**
2. Switch to the **Preview** tab
3. Add `PUBLIC_PAYLOAD_URL` pointing to your staging Payload backend (or the same production backend if you only have one)
4. Add `SITE_URL` = `https://staging.{project}.pages.dev`

The Production tab should already have the production values from Step 6.

### Verify staging works

1. Push to the staging branch: `git checkout staging && git merge main && git push`
2. Wait for the Cloudflare build to complete
3. Visit `staging.{project}.pages.dev` - the site should load
4. Check `staging.{project}.pages.dev/robots.txt` - should show `Disallow: /` (auto-noindex for staging URLs)

### Branch workflow

| Branch | Deploys to | Purpose |
|--------|-----------|---------|
| `main` | Production domain | Live site |
| `staging` | `staging.{project}.pages.dev` | Testing before production |
| Feature branches | Auto-generated preview URLs | PR review |

Merge flow: `feature/my-change` -> `staging` (test) -> `main` (production)

## Step 11: Enable CI Workflows

The template includes two GitHub Actions workflows that run on PRs and staging pushes.

### Add GitHub Actions secret

1. Go to your GitHub repo **Settings > Secrets and variables > Actions**
2. Click **New repository secret**
3. Name: `PAYLOAD_URL`, Value: your Payload backend URL (same as `PUBLIC_PAYLOAD_URL`)

### Lighthouse CI (`.github/workflows/lighthouse.yml`)

Runs on pushes to `staging` and PRs to `main`/`staging`. Tests performance, accessibility, best practices, and SEO with minimum thresholds (Performance >= 90, Accessibility >= 95, Best Practices = 100, SEO >= 95).

### Performance Budget (`.github/workflows/performance-budget.yml`)

Runs on the same triggers. Enforces:
- Total JS bundle < 50KB
- Any HTML page < 200KB

Fails the PR if budgets are exceeded. Writes a summary table to the GitHub Actions job summary.

### Verify CI works

Push a change to the staging branch or open a PR. Check the Actions tab in GitHub - both workflows should run and pass.

## Step 12: Custom Domain (Optional)

To add a custom domain:

1. Go to your Pages project **Custom domains** tab
2. Click **Set up a custom domain**
3. Enter your domain (e.g., `www.example.com`)
4. Cloudflare handles DNS and SSL automatically if the domain is already on Cloudflare

**Important:** If you have a Cloudflare zone-level Cache Rule (e.g., `s-maxage=604800` on `*.yourdomain.com`), it will cache HTML pages for up to 7 days. This prevents new deployments from being served. Either exclude the Pages subdomain from the cache rule or rely on the `_headers` file (already configured with `Cache-Control: public, max-age=0, must-revalidate` for HTML).

## Quick Reference

All settings at a glance for copy-pasting into the Cloudflare Pages dashboard:

```
Project name:           {project-name}
Production branch:      main
Framework preset:       None
Build command:          bun install && bun run build
Build output directory: (leave default - controlled by wrangler.jsonc)
Root directory:         frontend
Environment variable:   PUBLIC_PAYLOAD_URL = https://{your-payload-backend-url}
```

## How the Build Pipeline Works

The build process uses a strip/restore pattern to work around an incompatibility between `@astrojs/cloudflare` and Cloudflare Pages:

```
1. Cloudflare reads wrangler.jsonc          -> captures pages_build_output_dir: ./dist/client
2. scripts/strip-pages-config.mjs           -> removes pages_build_output_dir (avoids ASSETS binding error)
3. astro build                              -> builds static HTML to dist/client/
4. cloudflare-pages-build integration       -> cleans generated worker configs, patches CSP, fetches redirects
5. scripts/restore-pages-config.mjs         -> restores wrangler.jsonc, deletes deploy redirect
6. Cloudflare deploys from dist/client/     -> reads restored config, serves static files
```

**Why this is needed:** `@astrojs/cloudflare` generates worker configs that inherit `pages_build_output_dir` from the root `wrangler.jsonc`. This makes them "Pages configs", but they also contain an `ASSETS` binding that Cloudflare Pages rejects. Stripping the field before the build prevents the conflict. Cloudflare already captured the output directory before running the build command, so stripping it is safe.

### Key Files

| File | Purpose |
|------|---------|
| `frontend/wrangler.jsonc` | Cloudflare Pages config (output dir, compatibility flags) |
| `frontend/scripts/strip-pages-config.mjs` | Pre-build: strips pages_build_output_dir, saves backup |
| `frontend/scripts/restore-pages-config.mjs` | Post-build: restores config, deletes deploy redirect |
| `frontend/astro.config.mjs` | `cloudflare-pages-build` integration (cleanup, CSP patch, redirects) |
| `frontend/public/_headers` | Security headers + cache control for Cloudflare Pages |

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Build fails with "command not found: bun" | Verify Build system version is set to Version 3 (supports Bun natively) |
| Build fails with "ASSETS binding reserved in Pages projects" | The strip script didn't run. Check that `package.json` build command is `node scripts/strip-pages-config.mjs && astro build && node scripts/restore-pages-config.mjs` |
| Build succeeds but "deploy config redirect" error | The restore script didn't run. Check that `scripts/restore-pages-config.mjs` exists and deletes `.wrangler/deploy/config.json` |
| Build succeeds but pages are empty | Check that `PUBLIC_PAYLOAD_URL` is set correctly and the Payload backend is accessible |
| Build fails with module not found | The build command should be `bun install && bun run build` (includes install step) |
| Content not updating after Payload changes | Click "Deploy Website" in Payload admin, or push a commit to main |
| "Deploy Website" button doesn't trigger build | Check `CLOUDFLARE_DEPLOY_HOOK_URL` env var on the backend (Railway) |
| Custom domain shows old content for days | Zone-level cache rule likely caching HTML. Add cache exclusion for your Pages domain or purge the zone cache |
| 404 on page routes | Build output should be `dist/client` (set in wrangler.jsonc). If dashboard shows `dist`, update `pages_build_output_dir` in wrangler.jsonc |
| Build warns "Payload returned 403 for redirects" | The redirects collection needs `read: () => true` access (URL mappings are not sensitive) |
| Font 404 errors | Check that font files exist in `public/fonts/` and the paths in CSS match |
| Staging branch not deploying | Push at least one commit to `staging` branch. Cloudflare auto-deploys all branches by default |
| Staging shows same env vars as production | Set Preview-specific env vars in Cloudflare Pages Settings > Environment variables > Preview tab |
| Lighthouse CI not running | Add `PAYLOAD_URL` as GitHub Actions secret. Workflow triggers on `staging` push and PRs to `main`/`staging` |
| Performance budget failing | Check JS bundle size in `dist/client/_astro/*.js`. Budget is 50KB total JS, 200KB per HTML page |
