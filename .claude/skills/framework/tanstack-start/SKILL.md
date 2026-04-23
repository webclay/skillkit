---
name: tanstack-start
description: Use this skill when building with TanStack Start for type-safe routing, server functions, SSR, or RSC patterns. Activate when the user mentions TanStack Start, TanStack Router, createFileRoute, createServerFn, renderServerComponent, createCompositeComponent, Vinxi, RSC, React Server Components, SSR modes, or type-safe server functions.
---

# TanStack Start

Full-stack React framework with type-safe file-based routing, server functions, TanStack Query integration, and 5 rendering modes (CSR, SSR, data-only SSR, RSC low-level, RSC composite).

## When to Use This Skill

- User asks about TanStack Start or TanStack Router
- Project has `app.config.ts` with TanStack Start config
- User mentions type-safe routing or server functions
- Working with file-based routing in React
- User wants to use TanStack Query with SSR
- User asks about React Server Components (RSC) in TanStack Start
- User needs to choose between SSR, CSR, or RSC rendering modes

## Security Requirements

**IMPORTANT:** When setting up a new TanStack Start project or working with an existing one, you MUST also apply the [tanstack-nitro-security](../../security/tanstack-nitro-security/SKILL.md) skill to implement security headers middleware.

This is required for:
- New TanStack Start project setup
- Preparing for production deployment
- Security audits

The security skill adds Nitro middleware for HTTP security headers (X-Frame-Options, HSTS, etc.).

## Instructions

### Nitro Installation (Critical)

TanStack Start uses Nitro as its server layer. You **must** install the nightly version - this is the official recommendation from the TanStack Start docs.

**In `package.json`, add:**
```json
"nitro": "npm:nitro-nightly@latest"
```

Then install dependencies. Without this, Nitro will be imported but not actually installed, causing the dev server to fail.

### Vite Configuration (Critical)

The order of plugins in `vite.config.ts` is **critical**. Use this exact order:

```typescript
import tailwindcss from "@tailwindcss/vite";
import { tanstackStart } from "@tanstack/react-start/plugin/vite";
import viteReact from "@vitejs/plugin-react";
import { nitro } from "nitro/vite";
import { defineConfig } from "vite";
import viteTsConfigPaths from "vite-tsconfig-paths";

const config = defineConfig({
  plugins: [
    tanstackStart({ srcDirectory: "app" }),  // Must be FIRST
    nitro(),                                  // Must be SECOND
    viteTsConfigPaths({ projects: ["./tsconfig.json"] }),
    tailwindcss(),
    viteReact(),
  ],
  ssr: {
    external: ["lucide-react"],
    noExternal: [],
  },
  build: {
    minify: "esbuild",
    chunkSizeWarningLimit: 1000,
  },
});

export default config;
```

**Why order matters:** `tanstackStart()` must come before `nitro()` for proper server-side environment variable handling. Wrong order causes "No database host or connection string was set" errors.

### Nitro Configuration File (Required)

Create `nitro.config.ts` at project root. This file is **required** for Nitro to properly load `.env` files in server-side code:

```typescript
import { defineNitroConfig } from "nitro/config";

export default defineNitroConfig({});
```

Even an empty config triggers Nitro's automatic `.env` loading. Without this file, environment variables won't be available in server-side code.

### Environment Variables

- Use `.env` at project root (not `.env.local`)
- Nitro automatically loads `.env` when `nitro.config.ts` exists
- No need for `dotenv/config` in application code

### Project Structure

```
app/
├── routes/
│   ├── __root.tsx         # Root layout
│   ├── index.tsx          # Home (/)
│   ├── about.tsx          # /about
│   ├── posts/
│   │   ├── index.tsx      # /posts
│   │   └── $postId.tsx    # /posts/:postId
│   ├── _layout.tsx        # Pathless layout
│   └── api/
│       └── posts.ts       # API route
└── client.tsx
```

### Basic Route

```tsx
// app/routes/index.tsx
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/')({
  component: HomePage,
});

function HomePage() {
  return <h1>Welcome</h1>;
}
```

### Dynamic Route

```tsx
// app/routes/posts/$postId.tsx
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/posts/$postId')({
  component: PostDetail,
  loader: async ({ params }) => {
    const post = await getPost(params.postId);
    return { post };
  },
});

function PostDetail() {
  const { post } = Route.useLoaderData();
  return <h1>{post.title}</h1>;
}
```

### API Route (for Prisma/Auth)

```ts
// app/routes/api/posts.ts
import { createFileRoute } from '@tanstack/react-router';
import { db } from '@/lib/db';

export const Route = createFileRoute('/api/posts')({
  server: {
    handlers: {
      GET: async ({ request }) => {
        const posts = await db.post.findMany();
        return Response.json({ posts });
      },
      POST: async ({ request }) => {
        const body = await request.json();
        const post = await db.post.create({ data: body });
        return Response.json({ post });
      },
    },
  },
});
```

## Examples

**Root Layout:**
```tsx
// app/routes/__root.tsx
import { createRootRoute, Outlet } from '@tanstack/react-router';

export const Route = createRootRoute({
  component: () => (
    <html>
      <body>
        <Outlet />
      </body>
    </html>
  ),
});
```

**With Loader:**
```tsx
export const Route = createFileRoute('/dashboard')({
  component: Dashboard,
  pendingComponent: () => <div>Loading...</div>,
  errorComponent: ({ error }) => <div>Error: {error.message}</div>,
  loader: async () => {
    const data = await fetchDashboardData();
    return { data };
  },
});
```

**Search Params:**
```tsx
import { z } from 'zod';

export const Route = createFileRoute('/products')({
  component: Products,
  validateSearch: z.object({
    page: z.number().default(1),
    sort: z.enum(['name', 'date']).default('name'),
  }),
});

function Products() {
  const { page, sort } = Route.useSearch();
  const navigate = Route.useNavigate();

  return (
    <select
      value={sort}
      onChange={(e) => navigate({ search: { sort: e.target.value } })}
    >
      <option value="name">Name</option>
      <option value="date">Date</option>
    </select>
  );
}
```

**Navigation:**
```tsx
import { Link, useNavigate } from '@tanstack/react-router';

<Link to="/posts/$postId" params={{ postId: '123' }}>View Post</Link>
<Link to="/products" search={{ page: 2 }}>Page 2</Link>

const navigate = useNavigate();
navigate({ to: '/dashboard' });
```

**Protected Route:**
```tsx
// app/routes/_auth.tsx
export const Route = createFileRoute('/_auth')({
  beforeLoad: async () => {
    const session = await getSession();
    if (!session) {
      throw redirect({ to: '/login' });
    }
    return { session };
  },
  component: () => <Outlet />,
});
```

**Client Fetching API:**
```tsx
function ProductsPage() {
  const [products, setProducts] = useState([]);

  useEffect(() => {
    fetch('/api/products', { credentials: 'include' })
      .then(r => r.json())
      .then(d => setProducts(d.products));
  }, []);

  return <ul>{products.map(p => <li key={p.id}>{p.name}</li>)}</ul>;
}
```

## Rendering Modes (5 Options)

TanStack Start supports 5 rendering modes per route. All use the same loader pattern - only the route config changes:

| Mode | Config | Best For |
|------|--------|----------|
| CSR | `ssr: false` | SPAs, auth-gated pages where SEO doesn't matter |
| SSR (default) | `ssr: true` | Most pages - SEO, fast first paint |
| Data-only SSR | `ssr: 'data-only'` | Dashboards - secure data, no CDN-cached HTML |
| RSC Low-Level | `renderServerComponent()` | Server-only content, zero client JS |
| RSC Composite | `createCompositeComponent()` | Page shells with interactive client slots |

**RSC requires setup** - add the Vite RSC plugin and enable `rsc: true` on the TanStack Start plugin. See [rendering-modes.md](rendering-modes.md) for full examples.

**Key difference between SSR and RSC:** SSR renders on the server, then the client re-renders during hydration (standard React hydration cycle). RSC components only render on the server and are sent to the client as-is - no hydration re-render.

## Reference Files

- [rendering-modes.md](rendering-modes.md) - All 5 rendering modes with examples, decision tree, and RSC setup
- [data-fetching.md](data-fetching.md) - Data fetching strategy, loaders vs TanStack Query, preventing UI flash, caching
- [routing.md](routing.md) - Advanced routing patterns
- [server-functions.md](server-functions.md) - Server function patterns

## Critical: API Routes vs Server Functions

**Use API routes** (`/api/*.ts`) for Prisma and Better Auth:
- Route files with Prisma imports can leak to client bundle
- API routes are purely server-side

```ts
// CORRECT: API route for DB operations
// app/routes/api/users.ts
export const Route = createFileRoute('/api/users')({
  server: {
    handlers: {
      GET: async () => {
        const users = await db.user.findMany();
        return Response.json({ users });
      },
    },
  },
});
```

## Performance: FastResponse (Node.js)

If deploying to Node.js with Nitro, you can get a ~5% throughput improvement by replacing the global Response constructor with srvx's optimized FastResponse.

**Install srvx:**
```bash
bun add srvx
```

**Add to `src/server.ts` (or `app/server.ts`):**
```typescript
import { FastResponse } from "srvx";
globalThis.Response = FastResponse;
```

This works because srvx's FastResponse includes an optimized `_toNodeResponse()` path that avoids the overhead of standard Web Response to Node.js conversion. Only applies to Node.js deployments using Nitro/h3/srvx.

## Tips

- Use `Route.useParams()`, `Route.useSearch()`, `Route.useLoaderData()`
- API routes at `/api/*` are safe for Prisma/auth
- Use `credentials: 'include'` when fetching from client
- Validate search params with Zod schemas

## How to Verify

### Quick Checks
- `npm run build` - Build completes, route tree generates correctly
- `npm run dev` - Dev server starts without errors
- Check `routeTree.gen.ts` shows new routes

### Manual Verification
- Navigate to new route - page loads with correct data
- Dynamic routes resolve params correctly
- API routes return expected data (`curl localhost:3000/api/...`)
- Loaders fetch data before page renders

### Common Issues
- Route not found: Check file naming matches route path exactly
- Loader data undefined: Ensure loader returns data, use `Route.useLoaderData()`
- Type errors on params: Regenerate route tree with `npm run dev`
- Prisma in client bundle: Move DB code to API routes, not loaders
- "No database host or connection string was set": Missing `nitro.config.ts` or wrong Vite plugin order (tanstackStart must come before nitro)
- Environment variables undefined in server code: Add `nitro.config.ts` at project root
