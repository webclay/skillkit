---
name: cloudflare-pages
description: Use this skill when deploying an Astro + Payload CMS monorepo to Railway (backend) and Cloudflare Pages (frontend). Activate when the user mentions deploying to Cloudflare Pages, Railway + Cloudflare, monorepo deployment, deploy hooks, or static frontend deployment.
---

# Astro + Payload CMS Monorepo Deployment

Deploy pattern for projects using the astro-payload-template: Payload CMS on Railway (Docker + PostgreSQL), media storage on Cloudflare R2, and Astro frontend on Cloudflare Pages (static).

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

All 10 variables needed for the Payload backend on Railway:

| Variable | Value | When to set |
|----------|-------|-------------|
| `DATABASE_URI` | `${{Postgres.DATABASE_URL}}` | Initial setup |
| `PAYLOAD_PUBLIC_SERVER_URL` | `https://{service}-production.up.railway.app` | Initial setup |
| `PAYLOAD_SECRET` | `openssl rand -hex 32` | Initial setup |
| `R2_BUCKET` | `{project}-media` | After R2 setup (Step 2) |
| `R2_ENDPOINT` | `https://{account-id}.r2.cloudflarestorage.com` | After R2 setup (Step 2) |
| `R2_PUBLIC_URL` | `https://pub-{hash}.r2.dev` or custom domain | After R2 setup (Step 2) |
| `R2_ACCESS_KEY_ID` | From R2 API token | After R2 setup (Step 2) |
| `R2_SECRET_ACCESS_KEY` | From R2 API token | After R2 setup (Step 2) |
| `CLIENT_URI` | `https://{project}.pages.dev` | After CF Pages setup (Step 4) |
| `CLOUDFLARE_DEPLOY_HOOK_URL` | Webhook URL | After deploy hook setup (Step 4) |

The `backend/.env.production` file in the template lists all these variables as a reference. Set the first 3 during initial Railway setup, R2 variables after Step 2, and the last 2 after Cloudflare Pages is created.

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

## Step 2: Cloudflare R2 Storage

### 2.1 Create R2 Bucket

1. Cloudflare dashboard > R2 Object Storage > Create bucket
2. Bucket name: `{project}-media` (lowercase, hyphens)
3. Location: Automatic (or choose region close to users)

### 2.2 Enable Public Access

1. Open the bucket > Settings > Public access
2. Enable R2.dev subdomain (quick option) or set up a custom domain
3. Note the public URL (e.g., `https://pub-{hash}.r2.dev` or your custom domain)
4. This URL becomes `R2_PUBLIC_URL`

### 2.3 Create R2 API Token

1. R2 overview > Manage R2 API Tokens > Create API Token
2. Token name: `Payload - {project}`
3. Permissions: Object Read & Write
4. Specify bucket: select the bucket from 2.1
5. Create and **save immediately** (credentials shown only once):
   - Access Key ID -> `R2_ACCESS_KEY_ID`
   - Secret Access Key -> `R2_SECRET_ACCESS_KEY`
6. Note the Account ID from the Cloudflare dashboard URL or R2 overview page
7. Endpoint: `https://{account-id}.r2.cloudflarestorage.com` -> `R2_ENDPOINT`

### 2.4 Set R2 Variables on Railway

```bash
railway variable set "R2_BUCKET={project}-media" --service {service-name}
railway variable set "R2_ENDPOINT=https://{account-id}.r2.cloudflarestorage.com" --service {service-name}
railway variable set "R2_PUBLIC_URL=https://pub-{hash}.r2.dev" --service {service-name}
railway variable set "R2_ACCESS_KEY_ID={access-key}" --service {service-name}
railway variable set "R2_SECRET_ACCESS_KEY={secret-key}" --service {service-name}
```

---

## Step 3: Cloudflare Pages (Frontend)

### 3.1 Create Pages Project

1. Workers & Pages > Create > Pages > Connect to Git
2. Select the repo

### 3.2 Build Settings

| Setting | Value |
|---------|-------|
| **Framework preset** | None |
| **Build command** | `bun install && bun run build` |
| **Build output directory** | `dist` |
| **Root directory** (advanced) | `frontend` |

### 3.3 Environment Variable

| Variable | Value |
|---------|-------|
| `PAYLOAD_URL` | `https://{service}-production.up.railway.app` |

This is the only variable needed. The code reads `PUBLIC_PAYLOAD_URL` from `.env.production` (committed to git) and `PAYLOAD_URL` from wrangler.jsonc. Both point to the same Railway backend URL.

### 3.4 wrangler.jsonc

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

### 3.5 .env.production

Also committed to git (no secrets here):

```
PUBLIC_PAYLOAD_URL=https://{service}-production.up.railway.app
PUBLIC_SITE_URL=https://your-domain.com
PUBLIC_SITE_NAME=Site Name
```

### 3.6 Security Headers

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

### 3.7 Verify

- Pages URL: `https://{project}.pages.dev`
- Check that content loads from CMS
- Check security headers with browser DevTools (Network tab > Response Headers)

---

## Step 4: Connect Services

### 4.1 Set CLIENT_URI on Railway

```bash
railway variable set "CLIENT_URI=https://{project}.pages.dev" --service {service-name}
```

This configures CORS so the Astro frontend can access the Payload API.

### 4.2 Create Deploy Hook

1. Cloudflare Pages project > Settings > Build & Deployments > Deploy hooks
2. Add hook: Name "Payload", Branch "main"
3. Copy the webhook URL
4. Set on Railway:

```bash
railway variable set "CLOUDFLARE_DEPLOY_HOOK_URL={webhook-url}" --service {service-name}
```

### 4.3 How the Deploy Hook Works

```
Admin clicks "Deploy Website" in Payload sidebar
  -> POST /api/deploy (authenticated, backend)
  -> fetch(CLOUDFLARE_DEPLOY_HOOK_URL, { method: 'POST' })
  -> Cloudflare Pages triggers new build
  -> Astro fetches latest content from Payload API during build
  -> Static site deployed with fresh content
```

The deploy endpoint (`backend/src/endpoints/deploy.ts`) requires authentication - only logged-in admin users can trigger deploys.

### 4.4 Verify End-to-End

- Media uploads: Upload an image in Payload admin, verify it's stored in R2 and loads via the public URL
- Deploy hook: Click "Deploy Website" in Payload admin, verify Cloudflare Pages build triggers
- Content flow: Create a test post with an image, trigger rebuild, verify it appears on the frontend

---

## Environment Variable Flow

```
Railway (backend)                    Cloudflare Pages (frontend)
├── DATABASE_URI                     ├── wrangler.jsonc
├── PAYLOAD_SECRET                   │   └── vars.PAYLOAD_URL ──────────┐
├── PAYLOAD_PUBLIC_SERVER_URL        ├── .env.production                │
├── CLIENT_URI (CORS)                │   └── PUBLIC_PAYLOAD_URL ────────┤
├── CLOUDFLARE_DEPLOY_HOOK_URL       └── Dashboard env var              │
├── R2_BUCKET                            └── PAYLOAD_URL ───────────────┤
├── R2_ENDPOINT                                                         v
├── R2_PUBLIC_URL                    frontend/src/lib/payload.ts reads it
├── R2_ACCESS_KEY_ID                 as: import.meta.env.PUBLIC_PAYLOAD_URL
└── R2_SECRET_ACCESS_KEY
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
3. [ ] Set initial env vars: DATABASE_URI, PAYLOAD_PUBLIC_SERVER_URL, PAYLOAD_SECRET
4. [ ] Verify admin panel loads
5. [ ] Create admin account

### Cloudflare R2

1. [ ] Create R2 bucket (`{project}-media`)
2. [ ] Enable public access (R2.dev subdomain or custom domain)
3. [ ] Create R2 API token (Object Read & Write, scoped to bucket)
4. [ ] Save Access Key ID and Secret Access Key
5. [ ] Set R2_* env vars on Railway (5 variables)
6. [ ] Regenerate Payload import map: `cd backend && pnpm run payload generate:importmap`
7. [ ] Commit the updated import map and redeploy
8. [ ] Test media upload in Payload admin

### Cloudflare Pages

1. [ ] Create Pages project from Git
2. [ ] Set build command: `bun install && bun run build`
3. [ ] Set output directory: `dist`
4. [ ] Set root directory: `frontend`
5. [ ] Set `PAYLOAD_URL` env var
6. [ ] Verify site loads with CMS content

### Connect Services

1. [ ] Set `CLIENT_URI` on Railway (Cloudflare Pages URL)
2. [ ] Create deploy hook in Cloudflare Pages settings
3. [ ] Set `CLOUDFLARE_DEPLOY_HOOK_URL` on Railway
4. [ ] Test: click "Deploy Website" in Payload admin
5. [ ] Verify Cloudflare Pages build triggers
6. [ ] Add R2 public domain to CSP `img-src` in `_headers`

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

### Railway: S3 storage plugin client component not in import map

After adding the S3/R2 storage plugin, Payload fails because the plugin's client component isn't registered in the import map. Regenerate it and commit:

```bash
cd backend && pnpm run payload generate:importmap
```

This updates `src/app/(payload)/importMap.js`. Commit the file and redeploy.

### R2 upload fails with 403

R2 API token doesn't have write permissions, or is scoped to the wrong bucket. Verify the token in Cloudflare dashboard > R2 > Manage R2 API Tokens.
