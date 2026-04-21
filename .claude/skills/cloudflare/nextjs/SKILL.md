---
name: cloudflare-nextjs
description: Deploy Next.js applications to Cloudflare Workers using the OpenNext adapter (@opennextjs/cloudflare). Covers setup, configuration, dev vs preview distinction, bindings, ISR with R2, and supported features. Trigger words - nextjs cloudflare, next cloudflare, next.js workers, nextjs cloudflare deploy, opennext cloudflare, deploy next to cloudflare.
---

# Next.js on Cloudflare Workers

Deploy Next.js applications to Cloudflare Workers using the OpenNext adapter (`@opennextjs/cloudflare`).

**Prerequisites:** Read [workers-core](../workers-core/SKILL.md) for wrangler, bindings, and deployment fundamentals. Read [framework/nextjs](../../framework/nextjs/SKILL.md) for Next.js App Router patterns.

## When to Use This Skill

- Deploying a Next.js app to Cloudflare Workers
- Migrating a Next.js app from Vercel to Cloudflare
- Using Next.js with Cloudflare bindings (KV, R2, D1)

## Note on Payload CMS

Payload CMS uses Next.js as its admin panel. However, Payload requires a full Node.js runtime and is better deployed on Railway (see `deployment/railway` skill). This skill is for standalone Next.js frontend applications, not Payload backends.

## New Project

```bash
npm create cloudflare@latest -- my-next-app --framework=next
```

Or auto-detect an existing project:

```bash
npx wrangler deploy
```

## Supported Features

Most Next.js features work on Cloudflare Workers via OpenNext:

- App Router and Pages Router
- Server-side Rendering (SSR)
- Static Site Generation (SSG)
- Incremental Static Regeneration (ISR)
- Server Actions
- Route Handlers
- React Server Components
- Streaming
- Middleware
- Image Optimization (via Cloudflare Images)
- Partial Prerendering (PPR)
- Composable Caching

**Not yet supported:** Node.js runtime in Middleware (Next.js 15.2+)

## Configure Existing Project

### 1. Install Dependencies

```bash
bun add -d @opennextjs/cloudflare wrangler
```

### 2. Create open-next.config.ts

```typescript
import { defineCloudflareConfig } from "@opennextjs/cloudflare";

export default defineCloudflareConfig();
```

This is where you configure caching and other OpenNext options.

### 3. Create wrangler.jsonc

```jsonc
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "name": "my-next-app",
  "main": ".open-next/worker.js",
  "compatibility_date": "2026-04-21",
  "compatibility_flags": ["nodejs_compat"],
  "assets": {
    "directory": ".open-next/assets",
    "binding": "ASSETS"
  },
  "observability": {
    "enabled": true
  }
}
```

**Critical:** `compatibility_date` must be `2024-09-23` or later, and `nodejs_compat` is mandatory.

### 4. Update package.json Scripts

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "preview": "opennextjs-cloudflare build && opennextjs-cloudflare preview",
    "deploy": "opennextjs-cloudflare build && opennextjs-cloudflare deploy",
    "cf-typegen": "wrangler types --env-interface CloudflareEnv cloudflare-env.d.ts"
  }
}
```

### Dev vs Preview

| Command | Runtime | Use for |
|---------|---------|---------|
| `npm run dev` | Node.js (Next.js dev server) | Fast iteration, HMR |
| `npm run preview` | workerd (Cloudflare Workers) | Integration testing, verifying production behavior |

Always run `preview` before deploying to catch workerd-specific issues.

## Accessing Bindings

### In Server Components and Route Handlers

```typescript
// app/api/data/route.ts
import { getCloudflareContext } from "@opennextjs/cloudflare";

export async function GET() {
  const { env } = await getCloudflareContext();
  const value = await env.MY_KV.get("key");
  return Response.json({ value });
}
```

### In Server Actions

```typescript
"use server";

import { getCloudflareContext } from "@opennextjs/cloudflare";

export async function saveData(formData: FormData) {
  const { env } = await getCloudflareContext();
  await env.DB.prepare("INSERT INTO items (name) VALUES (?)").bind(
    formData.get("name")
  ).run();
}
```

### Type Generation

```bash
wrangler types --env-interface CloudflareEnv cloudflare-env.d.ts
```

This creates a `cloudflare-env.d.ts` file with typed bindings.

## Adding Bindings

```jsonc
{
  "name": "my-next-app",
  "main": ".open-next/worker.js",
  "compatibility_date": "2026-04-21",
  "compatibility_flags": ["nodejs_compat"],
  "assets": {
    "directory": ".open-next/assets",
    "binding": "ASSETS"
  },
  "kv_namespaces": [
    { "binding": "MY_KV", "id": "<kv-namespace-id>" }
  ],
  "d1_databases": [
    { "binding": "DB", "database_name": "my-db", "database_id": "<id>" }
  ],
  "r2_buckets": [
    { "binding": "CACHE_BUCKET", "bucket_name": "my-cache" }
  ]
}
```

## ISR with R2 Caching

Auto-config sets up R2 caching for ISR if R2 is enabled on your account. Configure in `open-next.config.ts`:

```typescript
import { defineCloudflareConfig } from "@opennextjs/cloudflare";

export default defineCloudflareConfig({
  // Caching configuration for ISR
});
```

## Remote Bindings (Local Dev)

Connect to deployed Cloudflare resources during local development:

```typescript
// next.config.ts
import { initOpenNextCloudflareForDev } from "@opennextjs/cloudflare";

initOpenNextCloudflareForDev({
  experimental: { remoteBindings: true },
});
```

```jsonc
{
  "r2_buckets": [
    {
      "bucket_name": "my-bucket",
      "binding": "MY_BUCKET",
      "experimental_remote": true
    }
  ]
}
```

Then `next dev` proxies binding requests to the real deployed resources.

## Workers Builds (CI/CD)

Configure environment variables in the "Build Variables and secrets" section of Workers Builds. Both `NEXT_PUBLIC_*` and non-public vars must be set there for the build to access them.

## Common Gotchas

1. **Use `preview`, not `dev`, for testing Workers behavior** - `dev` runs in Node.js, `preview` runs in workerd
2. **compatibility_date must be >= 2024-09-23** - Older dates break Next.js on Workers
3. **nodejs_compat is mandatory** - Next.js requires Node.js API compatibility
4. **Build uses OpenNext, not Wrangler** - Use `opennextjs-cloudflare build` (not `next build`) for deployment
5. **Build output is `.open-next/`** - Not `.next/`. The `main` and `assets.directory` must point to `.open-next/`
6. **Workers Builds env vars** - Both public and non-public env vars must be configured in the "Build Variables and secrets" section
7. **Image optimization** - Uses Cloudflare Images automatically. Configure `remotePatterns` in `next.config.ts` for external image sources
8. **Upgrade adapter regularly** - `@opennextjs/cloudflare` is actively developed. Keep it up to date for latest feature support and security fixes

## How to Verify

- `npm run dev` starts Next.js dev server
- `npm run preview` builds and serves in workerd runtime
- Server components render correctly
- API routes and Server Actions work
- ISR pages regenerate correctly
- `npm run deploy` succeeds
- Production URL serves the app
- Image optimization works (test with `<Image>` component)
