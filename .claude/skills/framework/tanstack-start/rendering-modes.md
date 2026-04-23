# Rendering Modes

TanStack Start supports 5 rendering modes on a per-route basis. All modes use the same loader pattern - only the route config changes.

## Quick Reference

| Mode | Route Config | Data Fetched On | Rendered On | Use When |
|------|-------------|-----------------|-------------|----------|
| CSR | `ssr: false` | Client | Client | SPAs, auth-gated dashboards where SEO doesn't matter |
| SSR | `ssr: true` (default) | Server (hard nav) / Client (soft nav) | Server + client hydration | Most pages - SEO, fast first paint |
| Data-only SSR | `ssr: 'data-only'` | Server | Client only | Dashboards with secure data but no CDN-cached HTML |
| RSC Low-Level | `renderServerComponent()` | Server | Server only (no hydration) | Server-rendered content with zero client JS |
| RSC Composite | `createCompositeComponent()` | Server | Server shell + client slots | Page shells with interactive sections |

## Mode 1: Client-Side Rendering (CSR)

Set `ssr: false` on the route. The page is never rendered on the server - only the JavaScript bundle is sent to the client, and the client does all the work.

```tsx
export const Route = createFileRoute('/dashboard')({
  ssr: false,
  loader: async () => {
    const data = await fetchDashboardData();
    return { data };
  },
  component: Dashboard,
});

function Dashboard() {
  const { data } = Route.useLoaderData();
  return <DashboardContent data={data} />;
}
```

The loader runs on the client. The router waits for the loader to complete before rendering the page, so you still get the clean "data ready on render" pattern.

## Mode 2: Server-Side Rendering (SSR) - Default

`ssr: true` is the default. On hard navigation (page refresh, direct URL), the page is rendered on the server and hydrated on the client. On soft navigation (SPA link click), the loader runs client-side like CSR.

```tsx
export const Route = createFileRoute('/posts')({
  // ssr: true is the default, you can omit it
  loader: async () => {
    const posts = await fetchPosts();
    return { posts };
  },
  component: Posts,
});
```

The HTML includes the rendered content (visible in page source). The client then hydrates and re-renders - this is the standard React hydration cycle.

## Mode 3: Data-Only SSR

Set `ssr: 'data-only'`. The server fetches the data but does NOT render HTML. Data is sent to the client, which does all the rendering.

```tsx
export const Route = createFileRoute('/admin/analytics')({
  ssr: 'data-only',
  loader: async () => {
    const stats = await fetchAnalytics();
    return { stats };
  },
  component: Analytics,
});
```

You may see a brief flash as the client renders. The content won't appear in the page source HTML, but the data will.

**Best for:**
- Dashboards where you don't want CDN-cached HTML
- Pages that need secure server-side data fetching but client-only rendering
- Admin panels where SEO doesn't matter

## Mode 4: RSC Low-Level API

Uses `renderServerComponent()` to render async server components that only execute on the server. No hydration re-render on the client.

**Requires RSC setup in Vite config:**

```typescript
import { reactServerComponents } from "@tanstack/react-start/plugin/vite";

const config = defineConfig({
  plugins: [
    tanstackStart({ srcDirectory: "app" }),
    reactServerComponents({ rsc: true }),
    nitro(),
    // ... other plugins
  ],
});
```

**Route with RSC:**

```tsx
import { createFileRoute } from '@tanstack/react-router';
import { createServerFn, renderServerComponent } from '@tanstack/react-start';

// Async server component - runs only on the server
function PokemonList() {
  const pokemon = await fetchTopPokemon();
  return (
    <ul>
      {pokemon.map((p) => (
        <li key={p.name}>{p.name}</li>
      ))}
    </ul>
  );
}

const getPokemonRenderable = createServerFn().handler(async () => {
  const renderable = await renderServerComponent(<PokemonList />);
  return { PokemonList: renderable };
});

export const Route = createFileRoute('/pokemon')({
  loader: async () => await getPokemonRenderable(),
  component: PokemonPage,
});

function PokemonPage() {
  const { PokemonList } = Route.useLoaderData();
  return (
    <div>
      <h1>Pokemon</h1>
      {PokemonList}
    </div>
  );
}
```

The loader can render multiple RSCs and fetch other data in parallel. Mix and match freely.

## Mode 5: RSC Composite Components

Uses `createCompositeComponent()` to define a server-rendered shell with named slots where client-side interactive components can be inserted - without needing `"use client"` directives.

```tsx
import { createFileRoute } from '@tanstack/react-router';
import { createServerFn, createCompositeComponent } from '@tanstack/react-start';

// Async server component for data rows
async function PokemonRows() {
  const pokemon = await fetchTopPokemon();
  return pokemon.map((p) => <tr key={p.name}><td>{p.name}</td></tr>);
}

const getPokemonComposite = createServerFn().handler(async () => {
  return createCompositeComponent({
    component: ({ children }) => (
      <div>
        <h1>Pokemon</h1>
        <table>
          <tbody>
            <PokemonRows />
          </tbody>
        </table>
        <div>{children}</div>
      </div>
    ),
  });
});

export const Route = createFileRoute('/pokemon')({
  loader: async () => {
    const src = await getPokemonComposite();
    return { src };
  },
  component: PokemonPage,
});

function PokemonPage() {
  const { src } = Route.useLoaderData();
  return (
    <CompositeComponent src={src}>
      <p>This is a client-side interactive section</p>
    </CompositeComponent>
  );
}
```

Composite components can have multiple named slots (not just `children`). Use them for page shells, layouts with ad sections, call-to-action areas, or any pattern where the outer structure is server-rendered but inner sections need interactivity.

## Choosing the Right Mode

```
Need SEO / fast first paint?
├── Yes: Does the page have interactive server components?
│   ├── Yes: Need a page shell with client slots?
│   │   ├── Yes → RSC Composite (Mode 5)
│   │   └── No → RSC Low-Level (Mode 4)
│   └── No → SSR (Mode 2, default)
└── No: Need secure server-side data fetching?
    ├── Yes → Data-only SSR (Mode 3)
    └── No → CSR (Mode 1)
```

## Mixing Modes

You can use different modes on different routes in the same app. The loader pattern stays consistent across all modes - only the route config and what you return from the loader changes.

A single loader can combine RSC renderables with regular data:

```tsx
export const Route = createFileRoute('/products')({
  loader: async () => {
    const [renderable, categories] = await Promise.all([
      getProductListRenderable(),
      fetchCategories(),
    ]);
    return { ProductList: renderable, categories };
  },
  component: ProductsPage,
});
```
