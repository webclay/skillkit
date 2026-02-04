---
name: astro
description: Astro web framework for content-focused websites. Trigger words - astro, blog, documentation site, marketing page, static site, content collections, islands, partial hydration, mdx
---

# Astro

Fast, content-focused web framework with islands architecture and zero JavaScript by default.

## When to Use This Skill

- Content-heavy sites (blogs, docs, marketing)
- Need minimal JavaScript and fast performance
- Want to mix multiple UI frameworks (React, Vue, Svelte)
- Static site generation with optional SSR
- Building with Markdown/MDX content

## When NOT to Use

- Highly interactive web apps (use Next.js or SvelteKit)
- Real-time applications
- Complex client-side state management

## Setup

```bash
npm create astro@latest
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

## Content Collections

```typescript
// src/content.config.ts
import { defineCollection, z } from 'astro:content';
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
├── index.astro          → /
├── about.astro          → /about
├── blog/
│   ├── index.astro      → /blog
│   └── [slug].astro     → /blog/:slug (dynamic)
└── [...slug].astro      → catch-all route
```

## Configuration

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
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
```

## Tips

- Use `client:visible` for below-fold interactive components
- Keep most components static (no client directive)
- Content Collections provide type safety for Markdown
- Mix frameworks on the same page if needed
- Use `getStaticPaths()` for dynamic routes in static mode

## How to Verify

### Quick Checks
- `npm run dev` starts without errors
- Pages render at correct routes
- Content collections query correctly
- Framework components hydrate with client directives

### Common Issues
- "Cannot find module 'astro:content'": Run `astro sync` or restart dev server
- Component not interactive: Add a `client:*` directive
- TypeScript errors in content: Check schema matches frontmatter
- Build fails: Ensure `getStaticPaths` returns all dynamic routes
