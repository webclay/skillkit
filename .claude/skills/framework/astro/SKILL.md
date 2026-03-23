---
name: astro
description: Use this skill when building content-focused websites with Astro, including blogs, documentation sites, or marketing pages. Activate when the user mentions Astro, content collections, islands architecture, partial hydration, MDX, or static site generation.
---

# Astro 6

Fast, content-focused web framework with islands architecture and zero JavaScript by default. Requires **Node 22+**.

## When to Use This Skill

- Content-heavy sites (blogs, docs, marketing)
- Need minimal JavaScript and fast performance
- Want to mix multiple UI frameworks (React, Vue, Svelte)
- Static site generation with optional SSR
- Building with Markdown/MDX content
- Frontend for headless CMS (Payload, Contentful, etc.)

## When NOT to Use

- Highly interactive web apps (use Next.js or SvelteKit)
- Real-time applications
- Complex client-side state management

## Setup

```bash
npm create astro@latest
```

**Upgrade existing project:**
```bash
npx @astrojs/upgrade
```

## Project Structure

```
my-site/
├── src/
│   ├── pages/           # Routes (required)
│   ├── components/      # Astro & framework components
│   ├── layouts/         # Page layouts
│   ├── content/         # Content collections
│   └── styles/          # Global styles
├── public/              # Static assets (fonts, icons)
├── astro.config.mjs     # Astro configuration
└── package.json
```

## Astro Components

```astro
---
// Component Script (server-side JavaScript)
import Layout from '../layouts/Layout.astro';
import Card from '../components/Card.astro';

interface Props {
  title: string;
  description?: string;
}

const { title, description = 'Default description' } = Astro.props;

// Fetch data at build time
const posts = await fetch('https://api.example.com/posts').then(r => r.json());
---

<!-- Component Template (HTML with expressions) -->
<Layout title={title}>
  <h1>{title}</h1>
  <p>{description}</p>

  <ul>
    {posts.map(post => (
      <li><a href={`/blog/${post.slug}`}>{post.title}</a></li>
    ))}
  </ul>

  <Card title="Hello" />
</Layout>

<style>
  /* Scoped CSS by default */
  h1 {
    color: var(--accent);
  }
</style>
```

## Slots

```astro
---
// Card.astro
interface Props {
  title: string;
}
const { title } = Astro.props;
---

<div class="card">
  <h2>{title}</h2>
  <slot />  <!-- Default slot -->
  <footer>
    <slot name="footer">Default footer</slot>  <!-- Named slot -->
  </footer>
</div>
```

Usage:
```astro
<Card title="My Card">
  <p>Card content goes here</p>
  <div slot="footer">Custom footer</div>
</Card>
```

## Content Collections (Build-Time)

Standard collections are built at build time from local files. Use `z` from `astro/zod` (not `astro:content`).

```typescript
// src/content.config.ts
import { defineCollection } from 'astro:content';
import { z } from 'astro/zod';
import { glob } from 'astro/loaders';

const blog = defineCollection({
  loader: glob({ pattern: '**/*.md', base: './src/content/blog' }),
  schema: z.object({
    title: z.string(),
    pubDate: z.coerce.date(),
    description: z.string(),
    author: z.string(),
    draft: z.boolean().optional(),
    tags: z.array(z.string()).optional(),
  }),
});

export const collections = { blog };
```

### Query Content

```astro
---
// src/pages/blog/index.astro
import { getCollection } from 'astro:content';

const posts = await getCollection('blog', ({ data }) => {
  return data.draft !== true;  // Filter out drafts
});

// Sort by date
posts.sort((a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf());
---

<ul>
  {posts.map(post => (
    <li>
      <a href={`/blog/${post.id}`}>{post.data.title}</a>
      <time>{post.data.pubDate.toDateString()}</time>
    </li>
  ))}
</ul>
```

### Dynamic Routes from Content

```astro
---
// src/pages/blog/[id].astro
import { getCollection, render } from 'astro:content';

export async function getStaticPaths() {
  const posts = await getCollection('blog');
  return posts.map(post => ({
    params: { id: post.id },
    props: { post },
  }));
}

const { post } = Astro.props;
const { Content } = await render(post);
---

<article>
  <h1>{post.data.title}</h1>
  <time>{post.data.pubDate.toDateString()}</time>
  <Content />
</article>
```

## Live Content Collections

Fetch content at request time without rebuilds. Perfect for CMS content and API data that changes frequently.

```typescript
// src/live.config.ts
import { defineLiveCollection } from 'astro:content';
import { z } from 'astro/zod';
import { cmsLoader } from './loaders/my-cms';

const updates = defineLiveCollection({
  loader: cmsLoader({ apiKey: import.meta.env.MY_API_KEY }),
  schema: z.object({
    slug: z.string(),
    title: z.string(),
    excerpt: z.string(),
    publishedAt: z.coerce.date(),
  }),
});

export const collections = { updates };
```

### Query Live Content

```astro
---
import { getLiveEntry } from 'astro:content';

const { entry: update, error } = await getLiveEntry(
  'updates',
  Astro.params.slug,
);

if (error || !update) {
  return Astro.redirect('/404');
}
---

<h1>{update.data.title}</h1>
<p>{update.data.excerpt}</p>
<time>{update.data.publishedAt.toDateString()}</time>
```

**Key points:**
- Same APIs as build-time collections - `getCollection`, `getEntry`
- Both types coexist in the same project
- No rebuild needed when CMS content changes
- Requires SSR mode (`output: 'server'`)

## Built-in Fonts API

Astro 6 handles font downloading, caching, self-hosting, optimized fallbacks, and preload links automatically.

```javascript
// astro.config.mjs
import { defineConfig, fontProviders } from 'astro/config';

export default defineConfig({
  fonts: [
    {
      name: 'Roboto',
      cssVariable: '--font-roboto',
      provider: fontProviders.fontsource(),
    },
  ],
});
```

```astro
---
import { Font } from 'astro:assets';
---

<head>
  <Font cssVariable="--font-roboto" preload />
</head>

<style is:global>
  body {
    font-family: var(--font-roboto);
  }
</style>
```

**Key points:**
- Fonts are self-hosted automatically (no external requests at runtime)
- Generates optimized fallback fonts to reduce layout shift
- Use `preload` attribute for above-the-fold fonts

## Content Security Policy

Automatically hashes scripts and styles, generates CSP headers. Works for static and SSR pages.

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';

export default defineConfig({
  security: {
    csp: true,  // Basic - auto-hashes everything
  },
});
```

Advanced configuration:

```javascript
export default defineConfig({
  security: {
    csp: {
      algorithm: 'SHA-512',
      directives: [
        "default-src 'self'",
        "img-src 'self' https://images.cdn.example.com",
      ],
      styleDirective: { hashes: ['sha384-styleHash'] },
      scriptDirective: { hashes: ['sha384-scriptHash'] },
    },
  },
});
```

## Route Caching (Experimental)

Platform-agnostic caching for server-rendered pages with automatic dependency tracing.

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import { memoryCache } from 'astro/config';

export default defineConfig({
  experimental: {
    cache: { provider: memoryCache() },
  },
});
```

```astro
---
Astro.cache.set({
  maxAge: 120,  // Cache for 2 minutes
  swr: 60,      // Serve stale for 1 minute while revalidating
  tags: ['home'],
});
---

<html><body>Cached page</body></html>
```

Content-aware invalidation:

```astro
---
import { getEntry } from 'astro:content';
const product = await getEntry('products', Astro.params.slug);

// When the product changes, Astro invalidates this cached page automatically
Astro.cache.set(product);
---
```

## Framework Components (Islands)

Install a framework integration:

```bash
npx astro add react
# or: npx astro add vue, svelte, solid, preact
```

### Client Directives

```astro
---
import ReactCounter from '../components/Counter.jsx';
---

<!-- Static (no JS sent to client) -->
<ReactCounter />

<!-- Hydrate immediately on page load -->
<ReactCounter client:load />

<!-- Hydrate when browser is idle -->
<ReactCounter client:idle />

<!-- Hydrate when component enters viewport -->
<ReactCounter client:visible />

<!-- Hydrate on media query match -->
<ReactCounter client:media="(max-width: 768px)" />

<!-- Client-only rendering (no SSR) -->
<ReactCounter client:only="react" />
```

### React Component Example

```tsx
// src/components/Counter.tsx
import { useState } from 'react';

export default function Counter({ initialCount = 0 }) {
  const [count, setCount] = useState(initialCount);

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

## Layouts

```astro
---
// src/layouts/BaseLayout.astro
import { ViewTransitions } from 'astro:transitions';
import { Font } from 'astro:assets';

interface Props {
  title: string;
  description?: string;
}

const { title, description } = Astro.props;
---

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width" />
  <meta name="description" content={description} />
  <title>{title}</title>
  <Font cssVariable="--font-body" preload />
  <ViewTransitions />
</head>
<body>
  <nav>
    <a href="/">Home</a>
    <a href="/blog">Blog</a>
  </nav>

  <main>
    <slot />
  </main>

  <footer>
    <p>&copy; {new Date().getFullYear()}</p>
  </footer>
</body>
</html>
```

## File-Based Routing

```
src/pages/
├── index.astro          -> /
├── about.astro          -> /about
├── blog/
│   ├── index.astro      -> /blog
│   └── [slug].astro     -> /blog/:slug (dynamic)
└── [...slug].astro      -> catch-all route
```

## Configuration

```javascript
// astro.config.mjs
import { defineConfig, fontProviders } from 'astro/config';
import react from '@astrojs/react';
import tailwind from '@astrojs/tailwind';
import mdx from '@astrojs/mdx';

export default defineConfig({
  site: 'https://example.com',
  integrations: [
    react(),
    tailwind(),
    mdx(),
  ],
  output: 'static',  // or 'server' for SSR
  fonts: [
    {
      name: 'Inter',
      cssVariable: '--font-body',
      provider: fontProviders.fontsource(),
    },
  ],
  security: {
    csp: true,
  },
  markdown: {
    shikiConfig: {
      theme: 'github-dark',
    },
  },
});
```

## Common Integrations

```bash
# Styling
npx astro add tailwind

# Frameworks
npx astro add react
npx astro add vue
npx astro add svelte

# Content
npx astro add mdx

# Deployment
npx astro add vercel
npx astro add netlify
npx astro add cloudflare
```

## Cloudflare Support

The `@astrojs/cloudflare` adapter runs `workerd` runtime at every stage: development, prerendering, and production. Access platform bindings (KV, D1, R2, Durable Objects) directly via `cloudflare:workers` - no simulation layers needed.

## View Transitions (SPA-like Navigation)

Astro's View Transitions make multi-page sites feel like a single-page app - instant navigation, no full page reloads, smooth animations.

```astro
---
// src/layouts/BaseLayout.astro
import { ViewTransitions } from 'astro:transitions';
---

<html>
<head>
  <ViewTransitions />  <!-- Enables SPA-like navigation -->
</head>
<body>
  <slot />
</body>
</html>
```

**What this gives you:**
- Pages are still pre-built static HTML (no loading states needed)
- Navigation fetches the next page in the background and swaps content
- Browser doesn't do a full reload - feels instant
- CSS transitions animate between pages

### Prefetching (Zero-Wait Navigation)

Astro can prefetch pages before the user clicks, making navigation feel truly instant:

```javascript
// astro.config.mjs
export default defineConfig({
  prefetch: {
    prefetchAll: true,          // Prefetch all links on the page
    // Or more targeted:
    // defaultStrategy: 'hover', // Prefetch when user hovers a link
  },
});
```

**Prefetch strategies:**
- `hover` (default) - Prefetch when user hovers over a link
- `tap` - Prefetch on mousedown (just before click)
- `viewport` - Prefetch when link enters viewport
- `load` - Prefetch all links on page load

**Per-link control:**
```astro
<a href="/about" data-astro-prefetch="viewport">About</a>
<a href="/heavy-page" data-astro-prefetch="hover">Heavy Page</a>
```

**Why this matters for Astro + Payload:** Since Astro builds static HTML at build time, there's no data fetching at runtime. Combined with View Transitions and prefetching, users navigate between pages with zero loading states - the content is already there.

## Experimental Features

### Rust Compiler

New successor to Go-based `.astro` compiler with improved speed and diagnostics. Planned as default in a future major version.

```bash
npm install @astrojs/compiler-rs
```

```javascript
export default defineConfig({
  experimental: {
    rustCompiler: true,
  },
});
```

### Queued Rendering

Two-pass rendering strategy showing up to 2x faster rendering with improved memory efficiency. Planned as default in Astro v7.

```javascript
export default defineConfig({
  experimental: {
    queuedRendering: { enabled: true },
  },
});
```

## Tips

- Use `client:visible` for below-fold interactive components
- Keep most components static (no client directive)
- Content Collections provide type safety for Markdown
- Mix frameworks on the same page if needed
- Use `getStaticPaths()` for dynamic routes in static mode
- Enable View Transitions for SPA-like navigation feel
- Use prefetching with `hover` strategy for instant page loads
- Use the Fonts API instead of manually loading fonts
- Enable CSP for automatic security headers
- Use Live Collections for CMS content that changes without rebuilds

## Reference Files

- [headless-cms.md](headless-cms.md) - Complete Astro + Payload CMS integration (fetching, blocks, theming, deployment)

## How to Verify

### Quick Checks
- `npm run dev` starts without errors
- Pages render at correct routes
- Content collections query correctly
- Framework components hydrate with client directives
- Fonts load without external requests (self-hosted)

### Common Issues
- "Cannot find module 'astro:content'": Run `astro sync` or restart dev server
- Component not interactive: Add a `client:*` directive
- TypeScript errors in content: Check schema matches frontmatter
- Build fails: Ensure `getStaticPaths` returns all dynamic routes
- Zod import errors: Use `import { z } from 'astro/zod'` (not `astro:content`)
- Node version errors: Astro 6 requires Node 22+
