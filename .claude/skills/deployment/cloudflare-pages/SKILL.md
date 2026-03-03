---
name: cloudflare-pages
description: Use this skill when deploying an Astro + Payload CMS monorepo to Railway (backend) and Cloudflare Pages (frontend). Activate when the user mentions deploying to Cloudflare Pages, Railway + Cloudflare, monorepo deployment, deploy hooks, or static frontend deployment.
---

# Astro + Payload CMS Monorepo Deployment

Deploy pattern for projects using the astro-payload-template: Payload CMS on Railway (Docker + PostgreSQL) and Astro frontend on Cloudflare Pages (static).

## Monorepo Structure

```
project/
├── package.json              # Root (dev scripts only)
├── backend/                  # Payload CMS (Next.js)
│   ├── Dockerfile            # Multi-stage Node 22 Alpine
│   ├── package.json          # pnpm, engines, start script
│   ├── next.config.mjs       # output: 'standalone'
│   ├── .env.production       # Template for Railway env vars
│   └── src/
│       ├── payload.config.ts # CORS, S3 storage, deploy endpoint
│       ├── endpoints/deploy.ts
│       └── components/DeployButton.tsx
├── frontend/                 # Astro (static)
│   ├── astro.config.mjs      # cloudflare adapter, output: 'static'
│   ├── wrangler.jsonc        # Cloudflare Pages config
│   ├── .env.production       # Build-time env vars
│   └── public/_headers       # Security headers
```

## Step 1: Railway (Backend)

### 1.1 Create Railway Project

1. New Project > Deploy from GitHub repo
2. Select the repo
3. Set **Root Directory** to `backend` (Settings > Source)

### 1.2 Add PostgreSQL

1. New Service > Database > PostgreSQL
2. Railway auto-provisions the database

### 1.3 Set Environment Variables

| Variable | Value |
|----------|-------|
| `DATABASE_URI` | `${{Postgres.DATABASE_URL}}` |
| `PAYLOAD_SECRET` | `openssl rand -hex 32` |
| `PAYLOAD_PUBLIC_SERVER_URL` | `https://{service}-production.up.railway.app` |
| `CLIENT_URI` | `https://{project}.pages.dev` (add after CF Pages setup) |
| `CLOUDFLARE_DEPLOY_HOOK_URL` | (add after CF Pages deploy hook setup) |

#### R2 Storage (if using file uploads)

| Variable | Value |
|----------|-------|
| `R2_BUCKET` | Bucket name |
| `R2_ENDPOINT` | `{account-id}.r2.cloudflarestorage.com` |
| `R2_PUBLIC_URL` | CDN domain for R2 |
| `R2_ACCESS_KEY_ID` | From R2 API token |
| `R2_SECRET_ACCESS_KEY` | From R2 API token |

### 1.4 How the Dockerfile Works

Multi-stage build (no configuration needed):

1. **deps** - Detects pnpm from lockfile, runs `pnpm i --frozen-lockfile`
2. **builder** - Copies deps + source, runs `pnpm run build` (Next.js)
3. **runner** - Copies standalone output, runs as non-root user on port 3000

**Critical**: `next.config.mjs` must have `output: 'standalone'`. The Dockerfile copies `.next/standalone` and `.next/static` for the production image.

### 1.5 Verify

- Backend URL: `https://{service}-production.up.railway.app/admin`
- Create first admin account
- Health check: `GET /api/globals/site-settings` returns JSON

---

## Step 2: Cloudflare Pages (Frontend)

### 2.1 Create Pages Project

1. Workers & Pages > Create > Pages > Connect to Git
2. Select the repo

### 2.2 Build Settings

| Setting | Value |
|---------|-------|
| **Framework preset** | None |
| **Build command** | `bun install && bun run build` |
| **Build output directory** | `dist` |
| **Root directory** (advanced) | `frontend` |

### 2.3 Environment Variable

| Variable | Value |
|---------|-------|
| `PAYLOAD_URL` | `https://{service}-production.up.railway.app` |

This is the only variable needed. The code reads `PUBLIC_PAYLOAD_URL` from `.env.production` (committed to git) and `PAYLOAD_URL` from wrangler.jsonc. Both point to the same Railway backend URL.

### 2.4 wrangler.jsonc

This file is committed to git and Cloudflare reads it automatically:

```jsonc
{
  "$schema": "node_modules/wrangler/config-schema.json",
  "name": "project-name",
  "compatibility_date": "2025-04-01",
  "compatibility_flags": ["nodejs_compat"],
  "pages_build_output_dir": "./dist",
  "vars": {
    "PAYLOAD_URL": "https://{service}-production.up.railway.app"
  }
}
```

### 2.5 .env.production

Also committed to git (no secrets here):

```
PUBLIC_PAYLOAD_URL=https://{service}-production.up.railway.app
PUBLIC_SITE_URL=https://your-domain.com
PUBLIC_SITE_NAME=Site Name
```

### 2.6 Security Headers

`frontend/public/_headers` is served by Cloudflare Pages automatically:

```
/*
  X-Content-Type-Options: nosniff
  X-Frame-Options: SAMEORIGIN
  Referrer-Policy: strict-origin-when-cross-origin
  X-DNS-Prefetch-Control: on
  Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=()
  Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdn.hugeicons.com; img-src 'self' data: https:; font-src 'self' https://fonts.gstatic.com https://cdn.hugeicons.com; connect-src 'self'; frame-ancestors 'self'; base-uri 'self'; form-action 'self'
```

Adjust `style-src`, `font-src`, and `img-src` per project (e.g., add R2 domain to `img-src` for uploaded images).

### 2.7 Verify

- Pages URL: `https://{project}.pages.dev`
- Check that content loads from CMS
- Check security headers with browser DevTools (Network tab > Response Headers)

---

## Step 3: Deploy Hook (CMS Triggers Frontend Rebuild)

### 3.1 Create Cloudflare Deploy Hook

1. Cloudflare Pages project > Settings > Build & Deployments > Deploy hooks
2. Add hook: Name "Payload", Branch "main"
3. Copy the webhook URL

### 3.2 Add to Railway

Set `CLOUDFLARE_DEPLOY_HOOK_URL` in Railway with the webhook URL.

### 3.3 How It Works

```
Admin clicks "Deploy Website" in Payload sidebar
  -> POST /api/deploy (authenticated, backend)
  -> fetch(CLOUDFLARE_DEPLOY_HOOK_URL, { method: 'POST' })
  -> Cloudflare Pages triggers new build
  -> Astro fetches latest content from Payload API during build
  -> Static site deployed with fresh content
```

The deploy endpoint (`backend/src/endpoints/deploy.ts`) requires authentication - only logged-in admin users can trigger deploys.

---

## Environment Variable Flow

```
Railway (backend)                    Cloudflare Pages (frontend)
├── DATABASE_URI                     ├── wrangler.jsonc
├── PAYLOAD_SECRET                   │   └── vars.PAYLOAD_URL ──────────┐
├── PAYLOAD_PUBLIC_SERVER_URL        ├── .env.production                │
├── CLIENT_URI (CORS)                │   └── PUBLIC_PAYLOAD_URL ────────┤
├── CLOUDFLARE_DEPLOY_HOOK_URL       └── Dashboard env var              │
└── R2_* (storage)                       └── PAYLOAD_URL ───────────────┤
                                                                        v
                                     frontend/src/lib/payload.ts reads it
                                     as: import.meta.env.PUBLIC_PAYLOAD_URL
```

**Why three places for the same URL?** Cloudflare Pages merges them with this priority: Dashboard > wrangler.jsonc > .env.production. Having `.env.production` in git gives a working default. The dashboard override lets you change it without code changes.

---

## Checklist for New Projects

### From Template

1. [ ] Clone astro-payload-template
2. [ ] Update `frontend/wrangler.jsonc` - project name + PAYLOAD_URL
3. [ ] Update `frontend/.env.production` - PUBLIC_PAYLOAD_URL
4. [ ] Update `backend/src/payload.config.ts` - CORS origins if custom domain

### Railway

1. [ ] Create project + PostgreSQL
2. [ ] Set root directory to `backend`
3. [ ] Set all environment variables
4. [ ] Verify admin panel loads
5. [ ] Create admin account

### Cloudflare Pages

1. [ ] Create Pages project from Git
2. [ ] Set build command: `bun install && bun run build`
3. [ ] Set output directory: `dist`
4. [ ] Set root directory: `frontend`
5. [ ] Set `PAYLOAD_URL` env var
6. [ ] Verify site loads with CMS content

### Deploy Hook

1. [ ] Create deploy hook in Cloudflare Pages settings
2. [ ] Add `CLOUDFLARE_DEPLOY_HOOK_URL` to Railway
3. [ ] Test: click "Deploy Website" in Payload admin
4. [ ] Verify Cloudflare Pages build triggers

### R2 Storage (optional)

1. [ ] Create R2 bucket in Cloudflare
2. [ ] Generate R2 API token
3. [ ] Set R2_* env vars in Railway
4. [ ] Test media upload in Payload admin
5. [ ] Add R2 public domain to CSP `img-src` in `_headers`

---

## Troubleshooting

### Railway: "No start command found"

Root directory not set to `backend`. Railway is reading the root package.json.

### Railway: "Module not found" during build

Dependencies in `package.json` but not in `pnpm-lock.yaml`. Run `cd backend && pnpm install` and commit the updated lockfile.

### Cloudflare: "Unterminated string literal"

Astro frontmatter has imports placed after runtime code. Move all `import` statements to the top of the frontmatter block (before any `const` or function calls).

### Cloudflare: Build succeeds but pages show fallback content

`PAYLOAD_URL` points to wrong backend or backend isn't reachable during build. Check Railway URL and ensure backend is deployed and running.

### Deploy button: "Deploy hook URL not configured"

`CLOUDFLARE_DEPLOY_HOOK_URL` not set in Railway. Add the webhook URL from Cloudflare Pages settings.

### Media images return 404

R2 env vars not set or R2 public URL not matching. Check `R2_PUBLIC_URL` matches the URL used in `generateFileURL`. Also ensure R2 bucket CORS allows the frontend domain.
