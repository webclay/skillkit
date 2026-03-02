# Payload CMS Deployment and Runtime Compatibility

Reference for deploying Payload CMS and runtime compatibility notes for Bun/Node.js.

## Deploying Payload on Railway

Railway is the simplest option for Payload with PostgreSQL. It provides managed Postgres with a persistent volume and auto-deploys from GitHub.

**Required environment variables (3 total):**

| Variable | Value | Purpose |
|----------|-------|---------|
| `DATABASE_URI` | `${{Postgres.DATABASE_URL}}` | Wires to Railway's managed Postgres via template syntax |
| `PAYLOAD_PUBLIC_SERVER_URL` | `https://${{RAILWAY_PUBLIC_DOMAIN}}` | Auto-resolves to the service's public domain |
| `PAYLOAD_SECRET` | `openssl rand -hex 32` | Unique per project - auth and encryption key |

**Monorepo setup:** Set root directory to `/backend` and configure Dockerfile builder pointing to `backend/Dockerfile`.

For the complete step-by-step Railway deployment guide (project creation, Postgres, GitHub connection, monorepo paths, domain setup), see the [Railway Payload deployment guide](../../workflow/project-setup/railway-payload-deploy.md).

## Deploying Payload on Cloudflare Workers

Payload has official support for Cloudflare Workers. This is the recommended deployment when using Astro on Cloudflare Pages - keep the entire stack on Cloudflare.

**What you get:**
- **Cloudflare Workers** - Serverless platform running Payload (paid plan, ~$5/month)
- **D1** - SQLite database with global read replicas (reads from nearest node)
- **R2** - Object storage for media/uploads
- **Wrangler** - CLI for local dev and deployment
- **OpenNext** - Converts Next.js (which Payload runs on) to deploy on Workers

### Quick Start (One-Click Template)

Payload provides an official one-click deploy template at `payloadcms/payload/templates/with-cloudflare-d1`. It creates a D1 database and R2 bucket automatically bound to your worker.

For manual setup, start from the template and customize.

### Required Packages

```bash
pnpm add @payloadcms/db-d1-sqlite @payloadcms/storage-r2 @opennextjs/cloudflare cross-env
```

### Wrangler Configuration

```jsonc
// wrangler.jsonc
{
  "$schema": "node_modules/wrangler/config-schema.json",
  "main": ".open-next/worker.js",
  "name": "my-app",
  "compatibility_date": "2025-08-15",
  "compatibility_flags": [
    "nodejs_compat",
    "global_fetch_strictly_public"
  ],
  "assets": {
    "directory": ".open-next/assets",
    "binding": "ASSETS"
  },
  "d1_databases": [{
    "binding": "D1",
    "database_id": "<your D1 database ID>",
    "database_name": "my-app",
    "remote": true
  }],
  "r2_buckets": [{
    "binding": "R2",
    "bucket_name": "my-app"
  }]
}
```

**Important:** Bindings are `D1` and `R2` (not connection strings). PAYLOAD_SECRET is set as a Cloudflare secret, not in wrangler.jsonc.

### Payload Config for Cloudflare

The config needs to resolve bindings differently for local dev (via Wrangler proxy) vs production (via OpenNext context):

```typescript
// src/payload.config.ts
import { sqliteD1Adapter } from '@payloadcms/db-d1-sqlite';
import { r2Storage } from '@payloadcms/storage-r2';
import { lexicalEditor } from '@payloadcms/richtext-lexical';
import { buildConfig } from 'payload';
import { getCloudflareContext } from '@opennextjs/cloudflare';

const isProduction = process.env.NODE_ENV === 'production';

// Resolve Cloudflare bindings (D1, R2) for both local and production
const cloudflare = isProduction
  ? await getCloudflareContext({ async: true })
  : await import('wrangler').then(({ getPlatformProxy }) =>
      getPlatformProxy({ environment: process.env.CLOUDFLARE_ENV })
    );

export default buildConfig({
  editor: lexicalEditor(),
  secret: process.env.PAYLOAD_SECRET || '',
  collections: [Users, Media], // Your collections
  db: sqliteD1Adapter({
    binding: cloudflare.env.D1, // Passes actual D1 binding object
    // readReplicas: 'first-primary', // Enable after turning on in CF dashboard
  }),
  plugins: [
    r2Storage({
      bucket: cloudflare.env.R2, // Passes actual R2 binding object
      collections: { media: true },
    }),
  ],
});
```

**Key detail:** The adapter receives the actual binding objects (`cloudflare.env.D1`, `cloudflare.env.R2`), not string names. Wrangler automatically creates local proxies for these during development.

### Package.json Scripts

```json
{
  "scripts": {
    "dev": "cross-env NODE_OPTIONS=--no-deprecation next dev",
    "build": "cross-env NODE_OPTIONS=\"--no-deprecation --max-old-space-size=8000\" next build",
    "deploy": "pnpm run deploy:database && pnpm run deploy:app",
    "deploy:app": "opennextjs-cloudflare build && opennextjs-cloudflare deploy",
    "deploy:database": "cross-env NODE_ENV=production PAYLOAD_SECRET=ignore payload migrate",
    "preview": "opennextjs-cloudflare build && opennextjs-cloudflare preview",
    "generate:types": "payload generate:types",
    "generate:types:cloudflare": "wrangler types --env-interface CloudflareEnv cloudflare-env.d.ts"
  }
}
```

### Local Development

```bash
# Authenticate with Cloudflare first
pnpm wrangler login

# Standard local dev - Wrangler auto-binds D1/R2 locally
pnpm dev

# Preview full Cloudflare build locally
pnpm preview
```

Wrangler automatically creates local proxies for D1 and R2 during `pnpm dev` - no manual setup needed.

### Deploy

```bash
# Create migration (after schema changes)
pnpm payload migrate:create

# Deploy database + app
pnpm run deploy
```

The deploy command runs migrations against the remote D1 database, then builds via OpenNext and deploys to Workers.

### Read Replicas

D1 supports global read replicas for faster reads from any region. Enable in two steps:

1. Add to your DB adapter config:
```typescript
db: sqliteD1Adapter({
  binding: cloudflare.env.D1,
  readReplicas: 'first-primary', // First query hits primary, subsequent reads hit nearest replica
}),
```

2. Enable read replicas in your D1 Cloudflare dashboard.

Writes always go to the primary region. Reads serve from the nearest replica.

**Real-world performance gains** (from Cloudflare's benchmarks with database in Eastern North America):

| Metric | Without replicas | With replicas | Improvement |
|--------|-----------------|---------------|-------------|
| P50 | 300ms | 120ms | -60% |
| P90 | 480ms | 250ms | -48% |
| P99 | 760ms | 550ms | -28% |

### Alternative: Postgres + Hyperdrive

If you prefer PostgreSQL over D1/SQLite, you can use Payload's official `@payloadcms/db-postgres` adapter with Cloudflare Hyperdrive for connection pooling:

```typescript
import { postgresAdapter } from '@payloadcms/db-postgres';
import { getCloudflareContext } from '@opennextjs/cloudflare';

const cloudflare = await getCloudflareContext({ async: true });

export default buildConfig({
  db: postgresAdapter({
    pool: {
      connectionString: cloudflare.env.HYPERDRIVE.connectionString,
      maxUses: 1, // Required - connections can't be shared across requests on Workers
    },
  }),
});
```

Hyperdrive maintains a connection pool across the Cloudflare network and adds a query cache. Configure it in your Cloudflare dashboard pointing to any Postgres provider (Neon, Supabase, etc.).

### Staging Environments

The template supports multiple environments via `CLOUDFLARE_ENV`:

```bash
# Deploy to staging
CLOUDFLARE_ENV=staging pnpm run deploy
```

Configure staging bindings in wrangler.jsonc under the `env` key with separate D1 databases and R2 buckets.

### Full Cloudflare Stack (Recommended for Astro + Payload)

When using both Astro and Payload on Cloudflare:

```
my-website/
├── cms/                    # Payload on Cloudflare Workers
│   ├── src/
│   │   ├── collections/
│   │   ├── blocks/
│   │   └── payload.config.ts
│   ├── wrangler.jsonc
│   └── package.json
│
├── web/                    # Astro on Cloudflare Pages
│   ├── src/
│   │   ├── pages/
│   │   ├── components/
│   │   └── lib/
│   └── package.json
```

**Environment variables:**

```env
# cms/.env (local)
PAYLOAD_SECRET=your-local-secret

# web/.env (local)
PAYLOAD_API_URL=http://localhost:3000/api
```

**Production (set in Cloudflare dashboard):**

Workers (Payload):
- `PAYLOAD_SECRET` - Secure random string (set via `wrangler secret put PAYLOAD_SECRET`)
- D1 and R2 bindings configured in wrangler.jsonc

Pages (Astro):
- `PAYLOAD_API_URL` - Your Payload Worker URL (e.g., `https://my-app.workers.dev/api`)

### Limitations

- **Paid Workers plan required** (~$5/month) due to bundle size exceeding free tier 3MB limit
- **GraphQL not fully supported** on Workers - use REST API (upstream workerd issue)
- **D1 access via bindings only** - no traditional connection strings
- **Logs disabled by default** - enable in Cloudflare dashboard (counts against quota)

### Cost Estimate (Cloudflare Stack)

| Service | Cost |
|---------|------|
| Workers (paid plan) | ~$5/month |
| D1 database | Free tier covers most CMS usage |
| R2 storage | Free for first 10GB |
| Pages (Astro) | Free |
| **Total** | **~$5/month** |

Significantly cheaper than Railway + MongoDB Atlas (~$28/month), with the bonus of global edge performance from read replicas.

## Runtime Compatibility (Bun / Node.js)

**Payload does NOT officially support Bun as a runtime.** Their test suite only runs against Node.js.

### What works with Bun

- **Bun as package manager** (`bun install`, `bun add`) - works fine
- **`bun run dev`** - works because Next.js uses Node.js under the hood
- **Payload CLI with `--disable-transpile`** - workaround for Bun runtime

### What breaks with Bun

- **Payload CLI commands under Bun runtime** - fails because Bun can't resolve the `tsx` transpiler Payload uses internally
- Error: `Cannot find module 'tsx://...'`

### Workaround: --disable-transpile flag

```bash
# Force Bun runtime with --disable-transpile
bunx --bun payload migrate --disable-transpile
bunx --bun payload run src/seed.ts --disable-transpile
```

This disables `tsx` and lets Bun handle TypeScript transpilation itself.

### Recommended approach

**Use Bun as package manager, Node.js as runtime for Payload:**

```bash
# Install dependencies (Bun - fast)
bun install

# Dev server (works - Next.js uses Node internally)
bun run dev

# Payload CLI commands (safe - uses Node via bun run, not bun runtime)
bun run payload migrate:create
bun run payload generate:types

# If you MUST use Bun runtime for CLI
bunx --bun payload migrate --disable-transpile
```

**Important:** This applies to the Payload backend only. The Astro frontend has full Bun support.

**User note:** Use Bun as the default package manager for everything (Astro, daily work, installing packages). Keep pnpm available in the Payload backend for CLI commands where Node.js runtime is required (migrations, type generation). The backend `package.json` keeps its pnpm engine config for this reason. From the root, `bun run dev:backend` works fine since it delegates to Next.js which uses Node internally.
