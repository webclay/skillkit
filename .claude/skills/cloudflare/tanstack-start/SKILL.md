---
name: cloudflare-tanstack-start
description: Deploy TanStack Start applications to Cloudflare Workers using the Cloudflare Vite Plugin. Covers setup, Vite configuration, wrangler.jsonc, accessing bindings via cloudflare:workers, custom entrypoints for queues/crons/durable objects, prerendering, and migration from Nitro. Trigger words - tanstack start cloudflare, tanstack start workers, tanstack cloudflare deploy, deploy tanstack to cloudflare.
---

# TanStack Start on Cloudflare Workers

Deploy TanStack Start full-stack applications to Cloudflare Workers using the Cloudflare Vite Plugin. This replaces Nitro entirely - on Workers you use the Cloudflare Vite Plugin instead.

**Prerequisites:** Read [workers-core](../workers-core/SKILL.md) for wrangler, bindings, and deployment fundamentals. Read [framework/tanstack-start](../../framework/tanstack-start/SKILL.md) for TanStack Start routing, server functions, and data fetching patterns.

## When to Use This Skill

- Deploying an existing TanStack Start app to Cloudflare Workers
- Creating a new TanStack Start project on Cloudflare
- Migrating a TanStack Start app from Nitro/Node.js to Cloudflare Workers
- Adding Cloudflare bindings (KV, R2, D1, AI) to a TanStack Start app

## Critical: Cloudflare Replaces Nitro

On Cloudflare Workers, the Cloudflare Vite Plugin replaces Nitro as the server layer. This means:

- **Remove** `nitro` and `nitro/vite` from dependencies
- **Remove** `nitro.config.ts`
- **Remove** `nitro()` from Vite plugins
- **Add** `@cloudflare/vite-plugin` and `wrangler`
- The Vite plugin order is different from Node.js/Nitro deployment

## New Project

```bash
npm create cloudflare@latest -- my-app --framework=tanstack-start
```

This scaffolds a TanStack Start project pre-configured for Workers with the Vite plugin, wrangler config, and correct scripts.

## Configure Existing Project

### 1. Install Dependencies

```bash
bun add -d @cloudflare/vite-plugin wrangler
```

If migrating from Nitro, also remove Nitro:

```bash
bun remove nitro
```

### 2. Update vite.config.ts

```typescript
import { defineConfig } from "vite";
import { tanstackStart } from "@tanstack/react-start/plugin/vite";
import { cloudflare } from "@cloudflare/vite-plugin";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [
    cloudflare({ viteEnvironment: { name: "ssr" } }),
    tanstackStart(),
    react(),
  ],
});
```

**Plugin order matters:**
1. `cloudflare()` - first (replaces Nitro's role)
2. `tanstackStart()` - second
3. `react()` - third

This is different from the Nitro setup where `tanstackStart()` comes first.

### 3. Create wrangler.jsonc

```jsonc
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "name": "my-app",
  "compatibility_date": "2026-04-21",
  "compatibility_flags": ["nodejs_compat"],
  "main": "@tanstack/react-start/server-entry",
  "observability": {
    "enabled": true
  }
}
```

The `main` field uses the TanStack Start server entry point directly.

### 4. Update package.json Scripts

```json
{
  "scripts": {
    "dev": "vite dev",
    "build": "vite build",
    "preview": "vite build && vite preview",
    "deploy": "vite build && wrangler deploy",
    "cf-typegen": "wrangler types"
  }
}
```

### 5. Remove Nitro Config

Delete `nitro.config.ts` if it exists. On Cloudflare Workers, Nitro is not used.

## Accessing Bindings

Import `env` from `cloudflare:workers` in server functions:

```typescript
import { createServerFn } from "@tanstack/react-start";
import { env } from "cloudflare:workers";

const getData = createServerFn().handler(async () => {
  // Access any binding configured in wrangler.jsonc
  const value = await env.MY_KV.get("key");
  const result = await env.DB.prepare("SELECT * FROM users").all();
  const aiResponse = await env.AI.run("@cf/meta/llama-3-8b-instruct", {
    prompt: "Hello",
  });

  return { value, users: result.results, ai: aiResponse };
});
```

### Type Safety

After configuring bindings in `wrangler.jsonc`, generate types:

```bash
wrangler types
```

This creates type definitions so `env.MY_KV`, `env.DB`, etc. are properly typed.

## Adding Bindings

Add bindings to `wrangler.jsonc` alongside the base config:

```jsonc
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "name": "my-app",
  "compatibility_date": "2026-04-21",
  "compatibility_flags": ["nodejs_compat"],
  "main": "@tanstack/react-start/server-entry",
  "kv_namespaces": [
    { "binding": "CACHE", "id": "<kv-namespace-id>" }
  ],
  "r2_buckets": [
    { "binding": "UPLOADS", "bucket_name": "my-uploads" }
  ],
  "d1_databases": [
    { "binding": "DB", "database_name": "my-db", "database_id": "<id>" }
  ],
  "ai": { "binding": "AI" }
}
```

## Custom Entrypoints

For advanced use cases (Queue consumers, Cron Triggers, Durable Objects, Workflows), create a custom server entry at `src/server.ts`:

```typescript
import handler from "@tanstack/react-start/server-entry";

// Export Durable Objects
export { MyDurableObject } from "./my-durable-object";

export default {
  // Delegate HTTP requests to TanStack Start
  fetch: handler.fetch,

  // Handle queue messages
  async queue(batch, env, ctx) {
    for (const message of batch.messages) {
      console.log("Processing:", message.body);
      message.ack();
    }
  },

  // Handle cron triggers
  async scheduled(event, env, ctx) {
    console.log("Cron fired:", event.cron);
  },
};
```

Then update `wrangler.jsonc` to point to your custom entry:

```jsonc
{
  "main": "src/server.ts",
  "queues": {
    "consumers": [
      { "queue": "my-queue", "max_batch_size": 10 }
    ]
  },
  "triggers": {
    "crons": ["0 * * * *"]
  }
}
```

## Static Prerendering

Supported from TanStack Start v1.138.0+. Add to Vite config:

```typescript
plugins: [
  cloudflare({ viteEnvironment: { name: "ssr" } }),
  tanstackStart({ prerender: { enabled: true } }),
  react(),
],
```

Prerendering uses local environment at build time. For production data in prerendered pages, use remote bindings.

## Environment Variables

On Cloudflare Workers, environment variables work differently from Nitro:

- **No `.env` auto-loading** - Workers doesn't use `dotenv`
- **Non-sensitive vars**: Put in `wrangler.jsonc` under `vars`
- **Secrets**: Use `wrangler secret put <NAME>` for production, `.dev.vars` for local dev
- **In server code**: Access via `import { env } from "cloudflare:workers"` (not `process.env`)

```jsonc
{
  "vars": {
    "PUBLIC_API_URL": "https://api.example.com"
  }
}
```

```
# .dev.vars (local development only, add to .gitignore)
DATABASE_URL=postgres://localhost:5432/mydb
API_KEY=dev-key-123
```

## CI/CD with Workers Builds

For CI environments, set `CLOUDFLARE_INCLUDE_PROCESS_ENV=true` to make environment variables available during the build step.

## Migration Checklist: Nitro to Cloudflare Workers

1. [ ] Install `@cloudflare/vite-plugin` and `wrangler`
2. [ ] Remove `nitro` package
3. [ ] Delete `nitro.config.ts`
4. [ ] Update `vite.config.ts` (replace `nitro()` with `cloudflare()`, reorder plugins)
5. [ ] Create `wrangler.jsonc`
6. [ ] Update `package.json` scripts
7. [ ] Replace `process.env` with `import { env } from "cloudflare:workers"` in server code
8. [ ] Move `.env` vars to `wrangler.jsonc` `vars` and `.dev.vars`
9. [ ] Run `wrangler types` for binding types
10. [ ] Test with `vite preview` (runs in workerd, not Node.js)
11. [ ] Deploy with `wrangler deploy`

## Common Gotchas

1. **Don't mix Nitro and Cloudflare Vite Plugin** - they serve the same purpose. Use one or the other.
2. **Plugin order** - `cloudflare()` must come before `tanstackStart()` and `react()`
3. **No process.env** - Use `import { env } from "cloudflare:workers"` in all server code
4. **dev vs preview** - `vite dev` runs in workerd via the Cloudflare plugin (production-accurate). `vite preview` builds first then runs in workerd. Both are accurate, but preview tests the actual build output.
5. **Prerendering with remote data** - Prerendered pages use local bindings at build time. Use remote bindings if you need production data during build.
6. **Security headers** - The Nitro-based `tanstack-nitro-security` skill does NOT apply on Cloudflare Workers. Use Cloudflare's built-in security features or add headers in your Worker code instead.

## How to Verify

- `vite dev` starts without errors and serves pages
- Server functions execute and return data
- Bindings are accessible (KV reads/writes work, D1 queries run)
- `vite build && vite preview` works (tests workerd runtime)
- `wrangler deploy` succeeds
- Production URL serves the app correctly
- `wrangler tail` shows request logs
