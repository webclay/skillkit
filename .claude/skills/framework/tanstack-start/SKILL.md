---
name: tanstack-start
description: TanStack Start framework patterns with type-safe routing and server functions. Trigger words - tanstack start, tanstack router, createFileRoute, createServerFn, server function, type-safe routing, vinxi
---

# TanStack Start

Full-stack React framework with type-safe file-based routing, server functions, and TanStack Query integration.

## When to Use This Skill

- User asks about TanStack Start or TanStack Router
- Project has `app.config.ts` with TanStack Start config
- User mentions type-safe routing or server functions
- Working with file-based routing in React
- User wants to use TanStack Query with SSR

## Instructions

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

## Reference Files

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
