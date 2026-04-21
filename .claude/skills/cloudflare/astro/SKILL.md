---
name: cloudflare-astro
description: Deploy Astro applications to Cloudflare Workers - both static (SSG) and server-rendered (SSR). Covers the @astrojs/cloudflare adapter, wrangler.jsonc configuration, accessing bindings, sessions with KV, and migration from Cloudflare Pages. Trigger words - astro cloudflare, astro workers, astro cloudflare deploy, astro cloudflare adapter, deploy astro to cloudflare.
---

# Astro on Cloudflare Workers

Deploy Astro sites to Cloudflare Workers - both fully static (SSG) and server-rendered (SSR) with access to Cloudflare bindings.

**Prerequisites:** Read [workers-core](../workers-core/SKILL.md) for wrangler, bindings, and deployment fundamentals. Read [framework/astro](../../framework/astro/SKILL.md) for Astro components, content collections, and islands architecture.

## When to Use This Skill

- Deploying an Astro site to Cloudflare Workers
- Migrating an Astro site from Cloudflare Pages to Workers
- Adding Cloudflare bindings (KV, R2, D1) to an Astro SSR site
- Setting up Astro + Payload CMS on Cloudflare Workers

## New Project

```bash
npm create cloudflare@latest -- my-astro-app --framework=astro
```

Or use automatic detection with an existing project:

```bash
npx wrangler deploy
```

Wrangler auto-detects Astro, installs the adapter, generates config, and deploys.

## Static Site (SSG)

For pre-rendered Astro sites with no server-side logic. No adapter needed.

### wrangler.jsonc

```jsonc
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "name": "my-astro-site",
  "compatibility_date": "2026-04-21",
  "assets": {
    "directory": "./dist"
  }
}
```

No `main` field needed. No `compatibility_flags` needed. Just point to the build output.

### package.json Scripts

```json
{
  "scripts": {
    "dev": "astro dev",
    "build": "astro build",
    "preview": "astro build && wrangler dev",
    "deploy": "astro build && wrangler deploy"
  }
}
```

## Server-Rendered Site (SSR)

For Astro sites with on-demand rendering, API routes, or Cloudflare bindings.

### 1. Install the Cloudflare Adapter

```bash
npx astro add cloudflare
```

This installs `@astrojs/cloudflare` and updates `astro.config.mjs` to set `output: 'server'`.

### 2. Install Wrangler

```bash
bun add -d wrangler
```

### 3. Create .assetsignore

Create `public/.assetsignore`:

```
_worker.js
_routes.json
```

This prevents the Worker script from being served as a static asset.

### 4. Create wrangler.jsonc

```jsonc
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "name": "my-astro-app",
  "main": "./dist/_worker.js/index.js",
  "compatibility_date": "2026-04-21",
  "compatibility_flags": ["nodejs_compat"],
  "assets": {
    "binding": "ASSETS",
    "directory": "./dist"
  },
  "observability": {
    "enabled": true
  }
}
```

### 5. Update package.json Scripts

```json
{
  "scripts": {
    "dev": "astro dev",
    "build": "astro build",
    "preview": "astro build && wrangler dev",
    "deploy": "astro build && wrangler deploy",
    "cf-typegen": "wrangler types"
  }
}
```

## Accessing Bindings

In SSR mode, access Cloudflare bindings via `Astro.locals`:

```astro
---
// src/pages/api/data.ts
const runtime = Astro.locals.runtime;
const value = await runtime.env.MY_KV.get("key");
---

<p>Value: {value}</p>
```

In API routes:

```typescript
// src/pages/api/users.ts
import type { APIRoute } from "astro";

export const GET: APIRoute = async ({ locals }) => {
  const { env } = locals.runtime;
  const result = await env.DB.prepare("SELECT * FROM users").all();
  return new Response(JSON.stringify(result.results), {
    headers: { "Content-Type": "application/json" },
  });
};
```

## Sessions

Astro on Cloudflare automatically uses Workers KV for sessions. A `SESSION` KV namespace is auto-provisioned.

## Prerender Control

In SSR mode (`output: 'server'`), all pages are server-rendered by default. Mark specific pages as static:

```astro
---
// src/pages/about.astro
export const prerender = true;
---

<h1>About</h1>
```

This is useful for pages that don't need dynamic rendering (privacy policy, about page, etc.).

## Adding Bindings

Add to `wrangler.jsonc`:

```jsonc
{
  "name": "my-astro-app",
  "main": "./dist/_worker.js/index.js",
  "compatibility_date": "2026-04-21",
  "compatibility_flags": ["nodejs_compat"],
  "assets": {
    "binding": "ASSETS",
    "directory": "./dist"
  },
  "kv_namespaces": [
    { "binding": "CACHE", "id": "<kv-namespace-id>" }
  ],
  "r2_buckets": [
    { "binding": "MEDIA", "bucket_name": "my-media" }
  ],
  "d1_databases": [
    { "binding": "DB", "database_name": "my-db", "database_id": "<id>" }
  ]
}
```

## Astro + Payload CMS on Workers

For content websites using Astro frontend + Payload CMS backend:

- **Backend (Payload):** Still deploys to Railway with PostgreSQL (Payload requires Node.js)
- **Frontend (Astro):** Deploys to Cloudflare Workers instead of Pages
- The deploy hook pattern works the same - Payload triggers a rebuild via webhook

The main differences from the old Pages deployment:
- Use `wrangler deploy` instead of `wrangler pages deploy`
- Use `wrangler.jsonc` with `assets.directory` instead of `pages_build_output_dir`
- The `*.workers.dev` subdomain replaces `*.pages.dev`

### Static Astro + Payload Config

For static builds (SSG) where Astro fetches content at build time:

```jsonc
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "name": "my-content-site",
  "compatibility_date": "2026-04-21",
  "assets": {
    "directory": "./dist"
  },
  "vars": {
    "PAYLOAD_URL": "https://backend-production.up.railway.app"
  }
}
```

### SSR Astro + Payload Config

For dynamic rendering where Astro fetches content per-request:

```jsonc
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "name": "my-content-site",
  "main": "./dist/_worker.js/index.js",
  "compatibility_date": "2026-04-21",
  "compatibility_flags": ["nodejs_compat"],
  "assets": {
    "binding": "ASSETS",
    "directory": "./dist"
  },
  "vars": {
    "PAYLOAD_URL": "https://backend-production.up.railway.app"
  }
}
```

## Migration from Cloudflare Pages

### For Static Sites

1. Replace `pages_build_output_dir: "./dist"` with `assets.directory: "./dist"` in wrangler config
2. Replace `wrangler pages deploy` with `wrangler deploy`
3. Update CI/CD scripts

### For SSR Sites

1. Ensure `@astrojs/cloudflare` adapter is installed
2. Create `.assetsignore` in `public/` with `_worker.js` and `_routes.json`
3. Update wrangler config to Workers format (see SSR section above)
4. Replace `wrangler pages deploy` with `wrangler deploy`

### Security Headers

On Pages, `public/_headers` was served automatically. On Workers, configure headers differently:

- For static sites: `public/_headers` still works with Workers static assets
- For SSR: Set headers in your Astro middleware or API routes

## Common Gotchas

1. **Missing .assetsignore** - Without it, `_worker.js` gets served as a static file instead of executing as the Worker
2. **nodejs_compat required for SSR** - Static sites don't need it, but SSR does
3. **Adapter sets output: 'server'** - Use `export const prerender = true` on pages that should be static
4. **Build output path** - The `main` field must point to `./dist/_worker.js/index.js` (not just `./dist/`)
5. **Node 22+ required** - Astro 6 requires Node 22+, make sure your CI environment matches
6. **Zod import** - Use `import { z } from 'astro/zod'` (not from `astro:content`)

## How to Verify

### Static Site
- `astro build` completes without errors
- `wrangler dev` serves the site locally
- All pages render correctly
- `wrangler deploy` succeeds

### SSR Site
- `astro dev` starts without errors (uses workerd runtime via adapter)
- Dynamic pages render with correct data
- Bindings work (KV reads, D1 queries)
- `astro build && wrangler dev` previews correctly in workerd
- `wrangler deploy` succeeds
- `wrangler tail` shows request logs
