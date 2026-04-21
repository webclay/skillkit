---
name: cloudflare-react-router
description: Deploy React Router v7 (formerly Remix) applications to Cloudflare Workers using the Cloudflare Vite Plugin. Covers setup, Worker entry point, accessing bindings via context.cloudflare.env, Durable Objects export, and limitations. Trigger words - react router cloudflare, react router workers, remix cloudflare, react router v7 deploy, react router cloudflare deploy.
---

# React Router on Cloudflare Workers

Deploy React Router v7 (formerly Remix) full-stack applications to Cloudflare Workers using the Cloudflare Vite Plugin.

**Prerequisites:** Read [workers-core](../workers-core/SKILL.md) for wrangler, bindings, and deployment fundamentals.

## When to Use This Skill

- Deploying a React Router v7 app to Cloudflare Workers
- Migrating a Remix app to React Router v7 on Cloudflare
- Building a full-stack React app with loaders/actions on Workers

## When NOT to Use

- SPA mode or prerendering - use [React + Vite](../react-vite/SKILL.md) with React Router as a library instead
- If you're using TanStack Start - use [cloudflare/tanstack-start](../tanstack-start/SKILL.md)

## New Project

```bash
npm create cloudflare@latest -- my-app --framework=react-router
```

Or auto-detect an existing project:

```bash
npx wrangler deploy
```

## Project Structure

```
my-app/
  app/
    routes/
      home.tsx          # Routes with loaders/actions
      _index.tsx
    root.tsx
    routes.ts
  workers/
    app.ts              # Worker entry point
  react-router.config.ts
  vite.config.ts
  wrangler.jsonc
```

## Configure Existing Project

### 1. Install Dependencies

```bash
bun add -d @cloudflare/vite-plugin wrangler
```

### 2. Update vite.config.ts

```typescript
import { defineConfig } from "vite";
import { cloudflare } from "@cloudflare/vite-plugin";
import { reactRouter } from "@react-router/dev/vite";
import tsconfigPaths from "vite-tsconfig-paths";

export default defineConfig({
  plugins: [
    cloudflare({ viteEnvironment: { name: "ssr" } }),
    reactRouter(),
    tsconfigPaths(),
  ],
});
```

### 3. Create Worker Entry (workers/app.ts)

```typescript
import { createRequestHandler } from "react-router";

const requestHandler = createRequestHandler(
  () => import("virtual:react-router/server-build"),
  import.meta.env.MODE
);

export default {
  async fetch(request: Request, env: Record<string, unknown>) {
    return requestHandler(request, { cloudflare: { env } });
  },
} satisfies ExportedHandler;
```

### 4. Update react-router.config.ts

```typescript
import type { Config } from "@react-router/dev/config";

export default {
  ssr: true,
  future: {
    v8_viteEnvironmentApi: true,
  },
} satisfies Config;
```

### 5. Create wrangler.jsonc

```jsonc
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "name": "my-app",
  "compatibility_date": "2026-04-21",
  "compatibility_flags": ["nodejs_compat"],
  "main": "./workers/app.ts",
  "assets": {
    "directory": "build/client"
  },
  "observability": {
    "enabled": true
  }
}
```

Auto-detect generates: `main: build/server/index.js`, `assets.directory: build/client`.

### 6. Update package.json Scripts

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

## Accessing Bindings

Access bindings via `context.cloudflare.env` in loaders and actions:

```tsx
// app/routes/dashboard.tsx
import type { Route } from "./+types/dashboard";

export async function loader({ context }: Route.LoaderArgs) {
  const env = context.cloudflare.env;

  const user = await env.DB.prepare(
    "SELECT * FROM users WHERE id = ?"
  ).bind(1).first();

  const cachedData = await env.CACHE.get("dashboard-stats");

  return { user, stats: cachedData ? JSON.parse(cachedData) : null };
}

export default function Dashboard({ loaderData }: Route.ComponentProps) {
  const { user, stats } = loaderData;
  return (
    <div>
      <h1>Welcome, {user.name}</h1>
    </div>
  );
}
```

In actions:

```tsx
export async function action({ request, context }: Route.ActionArgs) {
  const env = context.cloudflare.env;
  const formData = await request.formData();

  await env.DB.prepare(
    "INSERT INTO posts (title, body) VALUES (?, ?)"
  ).bind(formData.get("title"), formData.get("body")).run();

  return { success: true };
}
```

## Adding Bindings

```jsonc
{
  "name": "my-app",
  "compatibility_date": "2026-04-21",
  "compatibility_flags": ["nodejs_compat"],
  "main": "./workers/app.ts",
  "assets": {
    "directory": "build/client"
  },
  "d1_databases": [
    { "binding": "DB", "database_name": "my-db", "database_id": "<id>" }
  ],
  "kv_namespaces": [
    { "binding": "CACHE", "id": "<kv-namespace-id>" }
  ],
  "r2_buckets": [
    { "binding": "UPLOADS", "bucket_name": "my-uploads" }
  ]
}
```

## Exporting Durable Objects and Workflows

Export Durable Objects, Workflows, and other handlers directly from the Worker entry:

```typescript
// workers/app.ts
import { createRequestHandler } from "react-router";

const requestHandler = createRequestHandler(
  () => import("virtual:react-router/server-build"),
  import.meta.env.MODE
);

// Export Durable Objects
export { ChatRoom } from "../app/durable-objects/chat-room";
export { Counter } from "../app/durable-objects/counter";

export default {
  async fetch(request: Request, env: Record<string, unknown>) {
    return requestHandler(request, { cloudflare: { env } });
  },

  async queue(batch, env, ctx) {
    // Handle queue messages
  },

  async scheduled(event, env, ctx) {
    // Handle cron triggers
  },
} satisfies ExportedHandler;
```

## Limitations

- **SPA mode not supported** with the Cloudflare Vite Plugin - for SPA, use the [React + Vite](../react-vite/SKILL.md) template with React Router as a library
- **Prerendering not supported** with the Cloudflare Vite Plugin
- Runs in workerd during local dev (production-accurate, but some Node.js APIs may not be available)

## Common Gotchas

1. **`v8_viteEnvironmentApi: true` is required** in `react-router.config.ts` for the Cloudflare Vite Plugin to work
2. **Bindings via context, not import** - Unlike TanStack Start (`import { env } from "cloudflare:workers"`), React Router uses `context.cloudflare.env`
3. **Build output** - Client assets go to `build/client`, server bundle to `build/server`. The `assets.directory` must point to `build/client`
4. **No SPA mode** - If you need SPA, use React + Vite with React Router as a library, not as a framework

## How to Verify

- `vite dev` starts and serves routes with loaders
- Loaders fetch data from bindings correctly
- Actions write data via bindings
- Client-side navigation works
- `vite build && vite preview` runs in workerd
- `wrangler deploy` succeeds
- `wrangler tail` shows request logs
