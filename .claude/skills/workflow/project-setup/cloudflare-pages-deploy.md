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

The Payload backend must be live before connecting Cloudflare Pages, because the frontend needs `PAYLOAD_URL` as an environment variable to fetch content at build time.

**Deploy the backend based on your choice:**

| Platform | Guide |
|----------|-------|
| Railway | See [railway-payload-deploy.md](railway-payload-deploy.md) |
| Cloudflare Workers | See `cms/payload/deployment.md` (Cloudflare Workers section) |
| Other | Deploy Payload to your chosen platform and note the public URL |

After deployment, note the backend URL (e.g., `https://my-project-production.up.railway.app` or `https://my-app.workers.dev`).

## Step 4: Set Up Database

Database setup depends on your backend choice. If you followed the Railway guide, Postgres is already provisioned. If using Cloudflare Workers, D1 is configured via wrangler bindings.

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
| Build output directory | `dist` |
| Root directory (advanced) | `frontend` |

### Environment Variables

Click **+ Add variable** and add:

| Type | Name | Value |
|------|------|-------|
| Plaintext | `PAYLOAD_URL` | Your deployed Payload backend URL (e.g., `https://my-project-production.up.railway.app`) |

**Note:** The build command includes `bun install` because Cloudflare Pages needs to install the frontend dependencies within the `frontend/` root directory before building.

## Step 7: Save and Deploy

Click **Save and Deploy**. The first build starts immediately.

- Cloudflare Pages installs dependencies, runs the build, and deploys to its global CDN
- Your site will be available at `{project-name}.pages.dev`
- Automatic deployments are enabled by default - every push to `main` triggers a new build
- Pull requests get preview deployments automatically

## Step 8: Verify Post-Deploy Settings

After the first deploy completes, go to the Pages project **Settings** tab and verify:

### Build Settings

| Setting | Expected Value |
|---------|---------------|
| Git repository | Your GitHub repo |
| Build command | `bun install && bun run build` |
| Build output | `dist` |
| Root directory | `frontend` |
| Build system version | Version 3 |
| Build watch paths | Include paths: `*` |
| Build comments | Enabled |
| Branch control | Production: main, Automatic deployments: Enabled |

### Runtime Settings

These should be defaults, but verify they match:

| Setting | Value |
|---------|-------|
| Compatibility flags | `nodejs_compat` |
| Compatibility date | Recent date (e.g., Apr 1, 2025) |
| Fail open/closed | Fail open |
| Placement | Default |

### Variables and Secrets

| Type | Name | Value |
|------|------|-------|
| Plaintext | `PAYLOAD_URL` | Your Payload backend URL |

## Step 9: Custom Domain (Optional)

To add a custom domain:

1. Go to your Pages project **Custom domains** tab
2. Click **Set up a custom domain**
3. Enter your domain (e.g., `www.example.com`)
4. Cloudflare handles DNS and SSL automatically if the domain is already on Cloudflare

## Quick Reference

All settings at a glance for copy-pasting into the Cloudflare Pages dashboard:

```
Project name:           {project-name}
Production branch:      main
Framework preset:       None
Build command:          bun install && bun run build
Build output directory: dist
Root directory:         frontend
Environment variable:   PAYLOAD_URL = https://{your-payload-backend-url}
```

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Build fails with "command not found: bun" | Verify Build system version is set to Version 3 (supports Bun natively) |
| Build succeeds but pages are empty | Check that `PAYLOAD_URL` is set correctly and the Payload backend is accessible |
| Build fails with module not found | The build command should be `bun install && bun run build` (includes install step) |
| Preview deployments not working | Check Branch control settings - automatic deployments should be enabled |
| Content not updating after Payload changes | Astro builds are static - push a new commit or trigger a manual redeploy to pick up content changes |
| Custom domain not resolving | If using a non-Cloudflare domain, add the CNAME record pointing to `{project-name}.pages.dev` |
| 404 on page routes | Verify build output directory is `dist` (not `dist/client` or other) |

## Triggering Rebuilds on Content Changes

Since Astro generates static pages, content changes in Payload won't appear until the next build. Options:

1. **Manual redeploy** - Go to Pages dashboard and click "Retry deployment" on the latest deployment
2. **Deploy hook** - Create a deploy hook in Pages Settings > Build > Deploy Hooks, then call it from Payload's `afterChange` hook
3. **Push to main** - Any push to main triggers a rebuild automatically

For the deploy hook approach, add the webhook URL to Payload's backend environment as `CLOUDFLARE_DEPLOY_HOOK` and trigger it from a global `afterChange` hook.
