---
name: workers-core
description: Foundation skill for deploying to Cloudflare Workers. Covers wrangler CLI, wrangler.jsonc configuration, bindings (KV, R2, D1, Durable Objects, Queues, AI, Hyperdrive), environment variables, secrets, custom domains, static assets, Workers Builds CI/CD, and migration from Cloudflare Pages. Trigger words - cloudflare workers, wrangler, bindings, cloudflare deploy, workers deploy, cloudflare configuration, cloudflare setup, migrate pages to workers.
---

# Cloudflare Workers - Core

Foundation for deploying applications to Cloudflare Workers. All framework-specific Cloudflare skills reference this skill for shared concepts.

## When to Use This Skill

- Setting up a new Cloudflare Workers project
- Configuring bindings (KV, R2, D1, Durable Objects, Queues, AI)
- Managing environment variables and secrets
- Setting up custom domains or routes
- Migrating from Cloudflare Pages to Workers
- Understanding the Workers deployment model and pricing

## Tooling

### Wrangler CLI (v4+)

The primary CLI for Cloudflare Workers development and deployment.

```bash
bun add -d wrangler
```

Key commands:

| Command | Purpose |
|---------|---------|
| `wrangler dev` | Local dev server (port 8787, runs in workerd) |
| `wrangler deploy` | Build and deploy to Cloudflare |
| `wrangler deploy --dry-run` | Preview what would be deployed |
| `wrangler types` | Generate TypeScript types for bindings |
| `wrangler types --env-interface CloudflareEnv` | Generate types into a named interface |
| `wrangler secret put <NAME>` | Upload a secret value |
| `wrangler tail` | Stream live logs from production |
| `wrangler setup` | Configure project without deploying |

### Create Cloudflare CLI (C3)

Scaffolds new projects with framework-specific templates:

```bash
npm create cloudflare@latest -- my-app --framework=<framework>
```

Supported `--framework` values: `react`, `astro`, `next`, `react-router`, `tanstack-start`, `svelte`, `vue`, `nuxt`, `angular`, `solid`, `qwik`, `hono`

### Cloudflare Vite Plugin

First-class Vite integration that runs Worker code in `workerd` during development, matching production behavior exactly.

```bash
bun add -d @cloudflare/vite-plugin
```

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import { cloudflare } from "@cloudflare/vite-plugin";

export default defineConfig({
  plugins: [cloudflare()],
});
```

For full-stack frameworks (TanStack Start, React Router), assign to the SSR environment:

```typescript
plugins: [cloudflare({ viteEnvironment: { name: "ssr" } })]
```

### Automatic Configuration

Run `wrangler deploy` in any project without a wrangler config and it auto-detects the framework, generates `wrangler.jsonc`, and deploys. Use `--yes` for non-interactive CI mode. Requires wrangler 4.68.0+.

## Configuration (wrangler.jsonc)

Prefer `wrangler.jsonc` over `wrangler.toml` for comment support.

### Minimal Static Site

```jsonc
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "name": "my-app",
  "compatibility_date": "2026-04-21",
  "assets": {
    "directory": "./dist"
  }
}
```

### Full-Stack App

```jsonc
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "name": "my-app",
  "compatibility_date": "2026-04-21",
  "compatibility_flags": ["nodejs_compat"],
  "main": "./worker/index.ts",
  "assets": {
    "directory": "./dist",
    "binding": "ASSETS",
    "not_found_handling": "single-page-application"
  },
  "observability": {
    "enabled": true
  }
}
```

### Required Fields

| Field | Purpose |
|-------|---------|
| `name` | Worker name (becomes `<name>.<account>.workers.dev`) |
| `compatibility_date` | API version date (use today's date for new projects) |
| `compatibility_flags` | `["nodejs_compat"]` required for all SSR frameworks |
| `main` | Entry point for Worker code (omit for static-only sites) |

### Static Assets Configuration

| Field | Purpose |
|-------|---------|
| `assets.directory` | Path to built static files |
| `assets.binding` | Name to access assets from Worker code (`env.ASSETS.fetch(req)`) |
| `assets.not_found_handling` | `"none"` (default), `"single-page-application"`, or `"404-page"` |
| `assets.html_handling` | `"auto-trailing-slash"` (default), `"force-trailing-slash"`, `"drop-trailing-slash"`, `"none"` |
| `assets.run_worker_first` | `true`, `false`, or route patterns like `["/api/*", "!/api/docs/*"]` |

Default behavior: static assets are served first (free). Worker is invoked only for non-matching requests.

## Bindings

Bindings connect your Worker to Cloudflare platform resources. Configure in `wrangler.jsonc`:

### KV (Key-Value Store)

```jsonc
{
  "kv_namespaces": [
    { "binding": "MY_KV", "id": "<namespace-id>" }
  ]
}
```

Access: `env.MY_KV.get(key)`, `env.MY_KV.put(key, value)`

### R2 (Object Storage)

```jsonc
{
  "r2_buckets": [
    { "binding": "MY_BUCKET", "bucket_name": "my-bucket" }
  ]
}
```

Access: `env.MY_BUCKET.put(key, data)`, `env.MY_BUCKET.get(key)`

### D1 (SQL Database)

```jsonc
{
  "d1_databases": [
    { "binding": "DB", "database_name": "my-db", "database_id": "<id>" }
  ]
}
```

Access: `env.DB.prepare("SELECT * FROM users WHERE id = ?").bind(id).first()`

### Durable Objects

```jsonc
{
  "durable_objects": {
    "bindings": [
      { "name": "MY_DO", "class_name": "MyDurableObject" }
    ]
  }
}
```

Access: `env.MY_DO.get(env.MY_DO.idFromName("room-1"))`

### Queues

```jsonc
{
  "queues": {
    "producers": [
      { "binding": "MY_QUEUE", "queue": "my-queue" }
    ]
  }
}
```

Access: `env.MY_QUEUE.send({ type: "email", to: "user@example.com" })`

### Hyperdrive (PostgreSQL Proxy)

```jsonc
{
  "hyperdrive": [
    { "binding": "MY_DB", "id": "<hyperdrive-config-id>" }
  ]
}
```

Access: `env.MY_DB.connectionString` (use with any Postgres client)

### Workers AI

```jsonc
{
  "ai": { "binding": "AI" }
}
```

Access: `env.AI.run("@cf/meta/llama-3-8b-instruct", { prompt: "Hello" })`

### Service Bindings (Worker-to-Worker)

```jsonc
{
  "services": [
    { "binding": "AUTH_SERVICE", "service": "auth-worker" }
  ]
}
```

Access: `env.AUTH_SERVICE.fetch(request)`

### Vectorize

```jsonc
{
  "vectorize": [
    { "binding": "MY_INDEX", "index_name": "my-index" }
  ]
}
```

### Workflows

```jsonc
{
  "workflows": [
    { "binding": "MY_WORKFLOW", "name": "my-workflow", "class_name": "MyWorkflow" }
  ]
}
```

### Analytics Engine

```jsonc
{
  "analytics_engine_datasets": [
    { "binding": "AE", "dataset": "my-dataset" }
  ]
}
```

## Environment Variables and Secrets

### Non-Sensitive Variables

Commit to git in `wrangler.jsonc`:

```jsonc
{
  "vars": {
    "API_URL": "https://api.example.com",
    "APP_ENV": "production"
  }
}
```

### Secrets

Never commit secrets. Upload via CLI:

```bash
wrangler secret put DATABASE_URL
wrangler secret put API_KEY
```

For local development, create `.dev.vars` (add to `.gitignore`):

```
DATABASE_URL=postgres://localhost:5432/mydb
API_KEY=dev-key-123
```

### Declaring Required Secrets

```jsonc
{
  "secrets": {
    "required": ["DATABASE_URL", "API_KEY"]
  }
}
```

Validated at deploy time and used for type generation.

### Limits

- Free plan: 64 env vars per Worker
- Paid plan: 128 env vars per Worker
- Max 5 KB per variable

## Custom Domains and Routes

### Custom Domain (entire domain/subdomain)

Requires a Cloudflare-managed DNS zone:

```jsonc
{
  "routes": [
    { "pattern": "app.example.com", "custom_domain": true }
  ]
}
```

### Route Patterns (specific paths)

```jsonc
{
  "routes": [
    { "pattern": "example.com/api/*", "zone_name": "example.com" }
  ]
}
```

### workers.dev Subdomain

Every Worker gets `<name>.<account>.workers.dev` by default. Disable with:

```jsonc
{
  "workers_dev": false
}
```

## Deployment Model

1. **Build**: Framework builds client assets + server bundle (Worker script)
2. **Deploy**: `wrangler deploy` uploads both Worker code AND static assets as one unit
3. **Request routing at edge**:
   - Static asset match -> served directly (FREE, unlimited)
   - No match + Worker exists -> Worker invoked (billed)
   - Worker can call `env.ASSETS.fetch(request)` to serve assets programmatically

### Pricing

| | Free | Paid ($5/mo) |
|---|---|---|
| Static asset requests | Free | Free |
| Worker requests | 100,000/day | 10M included, $0.30/additional million |
| CPU time | 10ms | 30s (bundled) or 15ms (unbound) |
| Static files | 20,000 | 100,000 |
| Max file size | 25 MiB | 25 MiB |

## Workers Builds (CI/CD)

Connect GitHub/GitLab for automatic deployments:

1. Cloudflare dashboard > Workers & Pages > your Worker > Settings > Builds
2. Connect Git repo
3. Auto-detects framework and configures build
4. Preview URLs for non-production branches (enable `preview_urls: true`)

Build and runtime env vars are configured separately in Workers Builds.

## Migration from Cloudflare Pages

### Key Changes

| Pages | Workers |
|-------|---------|
| `wrangler pages deploy` | `wrangler deploy` |
| `wrangler pages dev` (port 8788) | `wrangler dev` (port 8787) |
| `pages_build_output_dir` | `assets.directory` |
| Auto SPA/404 detection | Explicit `not_found_handling` |
| `*.pages.dev` subdomain | `*.workers.dev` subdomain |
| ASSETS binding automatic | ASSETS binding manual |
| Preview env per branch | `preview_urls: true` |

### Migration Steps

1. Update Wrangler to v4+
2. Create `wrangler.jsonc` with `name`, `compatibility_date`, `assets.directory`
3. If SSR: set `main` field pointing to Worker script
4. Set `not_found_handling` explicitly
5. Replace deploy scripts (`wrangler pages deploy` -> `wrangler deploy`)
6. Update CI/CD configuration

### What Workers Has That Pages Doesn't

- Cloudflare Vite Plugin (workerd in dev)
- Native Durable Objects
- Cron Triggers
- Queue Consumers
- Email Workers
- Gradual Deployments
- Source Maps
- Workers Logs and Logpush

## Scripts Pattern

Standard `package.json` scripts for Workers projects:

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

## Common Gotchas

1. **nodejs_compat is mandatory** for all SSR frameworks. Set `compatibility_flags: ["nodejs_compat"]` and `compatibility_date` >= `2024-09-23`
2. **dev vs preview**: `npm run dev` may use Node.js; `wrangler dev` / `vite preview` uses workerd runtime. Always test with preview before deploy
3. **SPA routing**: Must explicitly set `not_found_handling: "single-page-application"` - Workers does NOT auto-detect this like Pages did
4. **Custom domains require Cloudflare-managed DNS zones** - domains must have their nameservers pointed to Cloudflare
5. **Secrets are not shared** between build time and runtime in Workers Builds - configure separately
6. **Free tier billing**: If requests match `run_worker_first` patterns and you exceed daily limits, you get 429 errors
7. **Configuration format**: Prefer `wrangler.jsonc` over `.toml` for comment support
8. **Type generation**: Run `wrangler types` after adding/changing bindings to get TypeScript types

## How to Verify

- `wrangler dev` starts without errors
- Bindings are accessible in Worker code
- `wrangler deploy --dry-run` shows correct configuration
- Static assets served at correct paths
- Custom domain resolves and serves the app
- `wrangler tail` shows request logs in production
