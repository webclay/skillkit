# Headless CMS Integration (Astro + Payload CMS)

Astro pairs excellently with Payload CMS as a headless backend. Payload handles content management and admin UI, while Astro renders blazing-fast static pages.

**Pattern:** Payload for backend - Astro fetches via REST API - Static build

**See:** `cms/payload` skill for complete Payload CMS setup and integration patterns.

## Fetching from Payload

```typescript
// src/lib/payload.ts
const PAYLOAD_API_URL = import.meta.env.PAYLOAD_API_URL || 'http://localhost:3000/api';

export async function getPage(slug: string) {
  const response = await fetch(
    `${PAYLOAD_API_URL}/pages?where[slug][equals]=${slug}&depth=2`
  );
  const data = await response.json();
  return data.docs[0];
}

export async function getAllPages() {
  const response = await fetch(`${PAYLOAD_API_URL}/pages`);
  const data = await response.json();
  return data.docs;
}
```

## Rendering Payload Blocks in Astro

Payload's blocks system maps perfectly to Astro components:

```astro
---
// src/pages/[slug].astro
import { getPage, getAllPages } from '../lib/payload';
import HeroBlock from '../components/blocks/HeroBlock.astro';
import ContentBlock from '../components/blocks/ContentBlock.astro';

export async function getStaticPaths() {
  const pages = await getAllPages();
  return pages.map((page) => ({
    params: { slug: page.slug },
    props: { page },
  }));
}

const { page } = Astro.props;
const { layout } = page;
---

<html>
  <head>
    <title>{page.title}</title>
  </head>
  <body>
    {layout.map((block) => {
      switch (block.blockType) {
        case 'hero':
          return <HeroBlock block={block} />;
        case 'content':
          return <ContentBlock block={block} />;
        default:
          return null;
      }
    })}
  </body>
</html>
```

## Block Components

```astro
---
// src/components/blocks/HeroBlock.astro
interface Props {
  block: {
    heading: string;
    subheading: any;
    image: { url: string; alt: string };
    ctaButton: { label: string; url: string };
  };
}

const { block } = Astro.props;
---

<section class="hero">
  <h1>{block.heading}</h1>
  <div set:html={block.subheading} />
  {block.image && <img src={block.image.url} alt={block.image.alt} />}
  {block.ctaButton && (
    <a href={block.ctaButton.url}>{block.ctaButton.label}</a>
  )}
</section>
```

## CMS-Driven Theming via CSS Variables

Store theme tokens (colors, fonts, spacing) as Payload fields and generate CSS variables in Astro at build time. This lets editors customize the site's look from the admin panel.

```typescript
// src/lib/payload.ts
export async function getTheme() {
  const response = await fetch(`${PAYLOAD_API_URL}/globals/theme?depth=0`);
  const data = await response.json();
  return data;
}
```

```astro
---
// src/layouts/BaseLayout.astro
import { getTheme } from '../lib/payload';

const theme = await getTheme();
const primaryColor = theme.primaryColor || '#3b82f6';
const secondaryColor = theme.secondaryColor || '#10b981';
const fontFamily = theme.fontFamily || 'Inter, sans-serif';
const borderRadius = theme.borderRadius || '0.5rem';
---

<html>
<head>
  <style define:vars={{ primaryColor, secondaryColor, fontFamily, borderRadius }}>
    /* These become CSS variables usable in any component */
    :root {
      --color-primary: var(--primaryColor);
      --color-secondary: var(--secondaryColor);
      --font-family: var(--fontFamily);
      --border-radius: var(--borderRadius);
    }
  </style>
</head>
<body>
  <slot />
</body>
</html>
```

All downstream components consume `var(--color-primary)` etc. - no prop drilling needed.

## Local API Caveat

Payload has a Local API example for Astro, but **the Local API does not work with Astro's SSG mode** due to `"__dirname is not defined in ES module scope"`. Stick with the REST API for static builds. The Local API may work in SSR mode but is not officially tested with Astro.

## Deployment

- **Payload:** Railway/Cloudflare Workers/Render/VPS (needs Node.js)
- **Astro:** Cloudflare Pages (recommended - free, global CDN) / Vercel / Netlify
- **Workflow:** Content changes in Payload - Trigger Astro rebuild - Deploy updated static site

**Recommended stack:** Payload on Railway (or Cloudflare Workers) + Astro on Cloudflare Pages. See `web-development/astro-payload-website/cloudflare-pages-deploy.md` for the full frontend deployment guide.

## When to Use This Pattern

**Use Astro + Payload when:**
- Content-heavy sites (marketing, blogs, documentation)
- Need admin UI for non-technical editors
- Want maximum performance (static generation)
- Content doesn't change constantly

**Don't use when:**
- Need real-time content updates
- Heavily interactive app (use Next.js + Payload instead)
- No admin panel needed (use Astro Content Collections)
