---
name: cloudflare-react-vite
description: Deploy React + Vite single-page applications (SPA) to Cloudflare Workers with an optional Worker API backend. Uses the Cloudflare Vite Plugin for development in workerd runtime. Trigger words - react cloudflare, react vite cloudflare, react spa workers, react workers deploy, vite cloudflare deploy.
---

# React + Vite on Cloudflare Workers

Deploy React single-page applications (SPA) to Cloudflare Workers with an optional backend API Worker. Uses the Cloudflare Vite Plugin for production-accurate local development.

**Prerequisites:** Read [workers-core](../workers-core/SKILL.md) for wrangler, bindings, and deployment fundamentals.

## When to Use This Skill

- Deploying a React SPA to Cloudflare Workers
- Building a React app with a Workers API backend
- Creating a client-rendered app with server-side API routes on Cloudflare
- Simple interactive apps that don't need SSR

## When NOT to Use

- Need SSR or server components - use [TanStack Start](../tanstack-start/SKILL.md) or [React Router](../react-router/SKILL.md) instead
- Full-stack framework with file-based routing - use TanStack Start or React Router
- Content-heavy static site - use [Astro](../astro/SKILL.md)

## New Project

```bash
npm create cloudflare@latest -- my-react-app --framework=react
```

This creates a React SPA with a Worker API backend, Cloudflare Vite Plugin, and wrangler config.

## Project Structure

```
my-react-app/
  src/
    App.tsx          # React SPA
    main.tsx         # React entry point
  worker/
    index.ts         # Worker API backend
  index.html         # HTML entry
  vite.config.ts     # Cloudflare Vite Plugin
  wrangler.jsonc     # Worker configuration
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
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [cloudflare(), react()],
});
```

No `viteEnvironment: { name: "ssr" }` needed - this is a SPA, not SSR.

### 3. Create Worker API (worker/index.ts)

```typescript
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);

    if (url.pathname === "/api/hello") {
      return Response.json({ message: "Hello from Workers!" });
    }

    if (url.pathname.startsWith("/api/")) {
      return new Response("Not found", { status: 404 });
    }

    // Static assets are handled by the assets configuration
    return new Response("Not found", { status: 404 });
  },
};
```

### 4. Create wrangler.jsonc

```jsonc
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "name": "my-react-app",
  "compatibility_date": "2026-04-21",
  "main": "worker/index.ts",
  "assets": {
    "directory": "./dist",
    "not_found_handling": "single-page-application"
  },
  "observability": {
    "enabled": true
  }
}
```

Key settings:
- `main` points to the Worker API backend
- `not_found_handling: "single-page-application"` routes unmatched paths to `index.html` for client-side routing
- Static assets are served from `./dist` (free, no Worker invocation)

### 5. Update package.json Scripts

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

## Asset Routing

The request flow:

1. Request arrives at Cloudflare edge
2. If path matches a static asset (JS, CSS, images) -> served for FREE
3. If no match and `not_found_handling` is `"single-page-application"` -> serves `index.html` (free, navigation request)
4. If `run_worker_first` matches -> Worker API handles the request (billed)

### Routing API Requests to Worker

To ensure API requests go to your Worker instead of the SPA fallback:

```jsonc
{
  "assets": {
    "directory": "./dist",
    "not_found_handling": "single-page-application",
    "run_worker_first": ["/api/*"]
  }
}
```

This makes `/api/*` routes always hit your Worker. All other routes serve the SPA.

## Accessing Bindings

React cannot directly access bindings. The React frontend calls the Worker API, and the Worker accesses bindings:

```typescript
// worker/index.ts
export default {
  async fetch(request: Request, env: Env): Promise<Response> {
    const url = new URL(request.url);

    if (url.pathname === "/api/users") {
      const result = await env.DB.prepare("SELECT * FROM users").all();
      return Response.json(result.results);
    }

    if (url.pathname === "/api/upload" && request.method === "POST") {
      const formData = await request.formData();
      const file = formData.get("file") as File;
      await env.UPLOADS.put(file.name, file.stream());
      return Response.json({ success: true });
    }

    return new Response("Not found", { status: 404 });
  },
};
```

```tsx
// src/App.tsx
function Users() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    fetch("/api/users")
      .then((r) => r.json())
      .then(setUsers);
  }, []);

  return (
    <ul>
      {users.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}
```

## Adding Bindings

```jsonc
{
  "name": "my-react-app",
  "compatibility_date": "2026-04-21",
  "main": "worker/index.ts",
  "assets": {
    "directory": "./dist",
    "not_found_handling": "single-page-application",
    "run_worker_first": ["/api/*"]
  },
  "d1_databases": [
    { "binding": "DB", "database_name": "my-db", "database_id": "<id>" }
  ],
  "r2_buckets": [
    { "binding": "UPLOADS", "bucket_name": "my-uploads" }
  ],
  "kv_namespaces": [
    { "binding": "CACHE", "id": "<kv-namespace-id>" }
  ]
}
```

## SPA-Only (No Worker Backend)

For a pure SPA with no server-side logic:

```jsonc
{
  "$schema": "./node_modules/wrangler/config-schema.json",
  "name": "my-spa",
  "compatibility_date": "2026-04-21",
  "assets": {
    "directory": "./dist",
    "not_found_handling": "single-page-application"
  }
}
```

No `main` field needed. All requests serve static assets or fall back to `index.html`. Zero cost (static assets are free).

## Common Gotchas

1. **Missing `not_found_handling`** - Without `"single-page-application"`, client-side routes return 404. This is the most common mistake.
2. **React can't access bindings** - Bindings are server-side only. Use fetch() to call your Worker API.
3. **No `nodejs_compat` needed for SPA-only** - Only add the flag if your Worker backend needs Node.js APIs.
4. **`run_worker_first` billing** - Requests matching these patterns invoke your Worker (billed). Unmatched routes serve free static assets.
5. **CORS not needed** - The Worker and SPA are on the same origin, so no CORS configuration required for API calls.

## How to Verify

- `vite dev` starts and serves the React app with HMR
- API routes respond correctly (`curl localhost:5173/api/hello`)
- Client-side routing works (navigate, refresh on a deep route)
- `vite build && vite preview` works in workerd runtime
- `wrangler deploy` succeeds
- Production URL serves the SPA and API correctly
