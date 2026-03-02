# Headless CMS Pattern (Payload + Astro)

Reference for using Payload as a headless CMS with Astro (or any frontend framework) - architecture, fetching, rendering blocks, deployment, and live preview.

**See also:** The `framework/astro` skill has complete patterns for using Astro as a frontend with Payload CMS backend.

## Architecture

**Payload Backend:**
- Collections, fields, relationships
- Admin panel at `/admin`
- REST API at `/api/*`
- Hosted separately or on the same server

**Astro Frontend:**
- Fetches content from Payload REST API
- Renders blocks as Astro components
- Fast, static-first builds

## Setup Pattern

**Directory structure:**

```
my-project/
├── payload/           # Payload CMS backend
│   ├── src/
│   │   ├── collections/
│   │   ├── blocks/
│   │   └── payload.config.ts
│   └── package.json
│
├── frontend/          # Astro frontend
│   ├── src/
│   │   ├── pages/
│   │   ├── components/
│   │   └── lib/
│   └── package.json
```

Or combined (Payload inside Astro project):

```
my-astro-site/
├── src/
│   ├── pages/         # Astro pages
│   ├── components/    # Astro components
│   └── payload/       # Payload CMS
│       ├── collections/
│       ├── blocks/
│       └── payload.config.ts
└── package.json
```

## Standard payload.config.ts (with Auto-Seeding)

Content website templates can include an `onInit` hook for automatic admin user seeding. This creates admin accounts on first startup so the CMS is immediately usable without manual registration.

The admin seeding config (email addresses, default password) is user-specific and defined in each project's `dev-context.md` under `## Admin Seeding`. The `onInit` hook reads email addresses and password from environment variables.

```typescript
import { buildConfig } from 'payload'
import { postgresAdapter } from '@payloadcms/db-postgres'
import { lexicalEditor } from '@payloadcms/richtext-lexical'
import path from 'path'
import { fileURLToPath } from 'url'
// ... import collections

const filename = fileURLToPath(import.meta.url)
const dirname = path.dirname(filename)

export default buildConfig({
  editor: lexicalEditor(),
  collections: [
    // ... your collections
  ],
  secret: process.env.PAYLOAD_SECRET || '',
  db: postgresAdapter({
    pool: {
      connectionString: process.env.DATABASE_URI || '',
    },
  }),
  // Auto-seed admin users on first start
  onInit: async (payload) => {
    const existingUsers = await payload.find({
      collection: 'users',
      limit: 1,
    })

    if (existingUsers.totalDocs === 0) {
      const adminEmails = (process.env.PAYLOAD_ADMIN_EMAILS || '').split(',').filter(Boolean)
      const password = process.env.PAYLOAD_ADMIN_PASSWORD || 'changeme123'

      for (const email of adminEmails) {
        await payload.create({
          collection: 'users',
          data: { email: email.trim(), password },
        })
      }

      if (adminEmails.length > 0) {
        payload.logger.info(`Seeded ${adminEmails.length} admin user(s)`)
      }
    }
  },
  typescript: {
    outputFile: path.resolve(dirname, 'payload-types.ts'),
  },
})
```

**Environment variables controlling auto-seeding:**

| Variable | Purpose | Default |
|----------|---------|---------|
| `PAYLOAD_ADMIN_EMAILS` | Comma-separated list of admin emails to create | (empty - no seeding) |
| `PAYLOAD_ADMIN_PASSWORD` | Password for all seeded admin accounts | `changeme123` |

**When it runs:** Only on first startup when the `users` collection is empty. Subsequent startups skip seeding entirely. See the auto-seeding section in [SKILL.md](SKILL.md) for full details.

**Where the values come from:** The setup wizard reads admin seeding preferences from `dev-context.md` and writes the appropriate values into the project's `.env` file. See [content-questionnaire.md](../../workflow/project-setup/content-questionnaire.md) for the wizard flow.

## Monorepo Development Workflow

**Local development with separate package.json files (simpler):**

```bash
# Terminal 1 - Run Payload
cd payload
npm run dev  # Runs on localhost:3000

# Terminal 2 - Run Astro
cd frontend
npm run dev  # Runs on localhost:4321, fetches from Payload
```

**Optional: Using workspaces (more advanced):**

Create `pnpm-workspace.yaml` at root:
```yaml
packages:
  - 'payload'
  - 'frontend'
```

Root `package.json`:
```json
{
  "name": "my-website-monorepo",
  "private": true,
  "scripts": {
    "dev:payload": "pnpm --filter payload dev",
    "dev:frontend": "pnpm --filter frontend dev",
    "dev": "concurrently \"pnpm dev:payload\" \"pnpm dev:frontend\""
  },
  "devDependencies": {
    "concurrently": "^8.2.2"
  }
}
```

Then run both with: `pnpm dev`

## Fetching Content from Payload in Astro

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
  const response = await fetch(`${PAYLOAD_API_URL}/pages?limit=1000`);
  const data = await response.json();
  return data.docs;
}
```

## Rendering Blocks in Astro

**Create Astro components for each block type:**

```astro
---
// src/components/blocks/HeroBlock.astro
import { RichTextRenderer } from '../RichTextRenderer.astro';

interface Props {
  block: {
    heading: string;
    subheading: any;
    image: {
      url: string;
      alt: string;
    };
    ctaButton: {
      label: string;
      url: string;
    };
  };
}

const { block } = Astro.props;
---

<section class="hero">
  <h1>{block.heading}</h1>
  <RichTextRenderer content={block.subheading} />
  {block.image && (
    <img src={block.image.url} alt={block.image.alt} />
  )}
  {block.ctaButton && (
    <a href={block.ctaButton.url} class="cta-button">
      {block.ctaButton.label}
    </a>
  )}
</section>
```

```astro
---
// src/components/blocks/ContentBlock.astro
import { RichTextRenderer } from '../RichTextRenderer.astro';

interface Props {
  block: {
    heading?: string;
    content: any;
  };
}

const { block } = Astro.props;
---

<section class="content">
  {block.heading && <h2>{block.heading}</h2>}
  <RichTextRenderer content={block.content} />
</section>
```

**Render blocks dynamically in Astro page:**

```astro
---
// src/pages/[slug].astro
import { getPage, getAllPages } from '../lib/payload';
import HeroBlock from '../components/blocks/HeroBlock.astro';
import ContentBlock from '../components/blocks/ContentBlock.astro';
import FormBlock from '../components/blocks/FormBlock.astro';

export async function getStaticPaths() {
  const pages = await getAllPages();
  return pages.map((page) => ({
    params: { slug: page.slug },
    props: { page },
  }));
}

const { page } = Astro.props;
const { layout } = page;

// Helper to render the right component for each block type
function getBlockComponent(blockType: string) {
  switch (blockType) {
    case 'hero':
      return HeroBlock;
    case 'content':
      return ContentBlock;
    case 'form':
      return FormBlock;
    default:
      return null;
  }
}
---

<html>
  <head>
    <title>{page.title}</title>
  </head>
  <body>
    <main>
      {layout.map((block) => {
        const Component = getBlockComponent(block.blockType);
        return Component ? <Component block={block} /> : null;
      })}
    </main>
  </body>
</html>
```

## Deployment

**Option 1: Full Cloudflare Stack (Recommended)**
- Payload on Cloudflare Workers (D1 database + R2 storage)
- Astro on Cloudflare Pages
- Single platform, ~$5/month total
- See deployment.md for full details

**Option 2: Railway + Cloudflare Pages**
- Payload on Railway (MongoDB/PostgreSQL)
- Astro on Cloudflare Pages
- ~$28/month

**Architecture: Monorepo (Recommended)**
- One repository with both projects (`cms/` + `web/`)
- Deploy each to different platforms from same repo

**Monorepo Structure for Separate Hosting:**

```
my-website/
├── payload/                 # Payload CMS backend
│   ├── src/
│   │   ├── collections/
│   │   ├── blocks/
│   │   └── payload.config.ts
│   ├── package.json
│   └── .env
│
├── frontend/                # Astro frontend
│   ├── src/
│   │   ├── pages/
│   │   ├── components/
│   │   └── lib/
│   ├── package.json
│   └── .env
│
└── README.md
```

**Railway (Payload Backend) Configuration:**

```
Root Directory: payload
Build Command: npm run build
Start Command: npm run serve
Environment Variables:
  DATABASE_URI=mongodb://...
  PAYLOAD_SECRET=your-secret-key
  CORS_ORIGIN=https://your-site.pages.dev
  NODE_ENV=production
```

**Cloudflare Pages (Astro Frontend) Configuration:**

```
Build command: npm run build
Build output directory: dist
Root directory: frontend
Environment Variables:
  PAYLOAD_API_URL=https://your-payload.railway.app/api
  NODE_VERSION=20
```

**Vercel/Netlify (Alternative for Astro):**

Same settings, just adjust platform-specific config files if needed.

**Environment variables:**

```env
# payload/.env (local)
DATABASE_URI=mongodb://localhost:27017/my-website
PAYLOAD_SECRET=your-local-secret
PORT=3000

# frontend/.env (local)
PAYLOAD_API_URL=http://localhost:3000/api
```

**Production environment variables (set in hosting platforms):**

Railway (Payload):
- `DATABASE_URI` - Production MongoDB/PostgreSQL URI
- `PAYLOAD_SECRET` - Secure random string
- `CORS_ORIGIN` - Your Astro site URL (e.g., `https://yoursite.pages.dev`)

Cloudflare Pages (Astro):
- `PAYLOAD_API_URL` - Your Payload Railway URL (e.g., `https://your-payload.railway.app/api`)

## Build Trigger Strategies

When using Astro in SSG mode, content changes in Payload need to trigger a site rebuild. Several approaches:

**Option 1: "Update Live Site" button in Payload admin**

Add a custom admin UI button that editors click when they're ready to publish:

```typescript
// Payload hook on a dedicated "site-settings" global
{
  slug: 'site-settings',
  hooks: {
    afterChange: [
      async () => {
        await fetch(process.env.ASTRO_BUILD_WEBHOOK, { method: 'POST' });
      }
    ]
  }
}
```

**Option 2: Debounced auto-rebuild**

Trigger a rebuild after a period of inactivity (e.g., no edits for 15 minutes). Implement with a simple timer in an afterChange hook:

```typescript
// Payload hook with debounce
let rebuildTimer: NodeJS.Timeout | null = null;

const triggerDebouncedRebuild = async () => {
  if (rebuildTimer) clearTimeout(rebuildTimer);
  rebuildTimer = setTimeout(async () => {
    await fetch(process.env.ASTRO_BUILD_WEBHOOK, { method: 'POST' });
  }, 15 * 60 * 1000); // 15 minutes
};

// Add to any collection that should trigger rebuilds
{
  slug: 'pages',
  hooks: {
    afterChange: [({ operation }) => { triggerDebouncedRebuild(); }],
  }
}
```

**Option 3: Direct webhook per change (simple)**

```typescript
// Payload hook to trigger Astro rebuild
{
  slug: 'pages',
  hooks: {
    afterChange: [
      async ({ doc }) => {
        await fetch(process.env.ASTRO_BUILD_WEBHOOK, {
          method: 'POST',
          headers: { 'Authorization': `Bearer ${process.env.REVALIDATE_SECRET}` },
          body: JSON.stringify({ slug: doc.slug }),
        });
      }
    ]
  }
}
```

Most hosting platforms (Vercel, Netlify, Cloudflare Pages) provide deploy hook URLs you can use as `ASTRO_BUILD_WEBHOOK`.

## Live Preview with Astro

Payload's live preview can work with Astro, though it requires SSR mode (not SSG).

**Basic approach: Full page refresh via postMessage**

Configure Payload to show your Astro page in its side panel preview frame:

```typescript
// payload.config.ts
{
  slug: 'pages',
  admin: {
    livePreview: {
      url: ({ data }) =>
        `${process.env.ASTRO_SITE_URL}/${data.slug}`,
    },
  },
}
```

On the Astro side, add a client-side script that listens for Payload's refresh signal:

```astro
---
// src/layouts/PreviewLayout.astro
---
<slot />

<script>
  // Listen for Payload's postMessage to refresh preview
  window.addEventListener('message', (event) => {
    if (event.data.type === 'payload-live-preview') {
      window.location.reload();
    }
  });
</script>
```

**Block-level preview (advanced)**

Create an Astro server route that renders a single block from query params, then embed it in Payload via iframe:

```typescript
// src/pages/api/preview-block.ts (Astro SSR route)
export async function GET({ url }) {
  const blockData = JSON.parse(url.searchParams.get('data') || '{}');
  const blockType = url.searchParams.get('type');
  // Render the block component and return HTML
}
```

**Note:** Live preview requires Astro in SSR mode (`output: 'server'` or `output: 'hybrid'`). For purely static sites, editors typically preview by triggering a build and checking the staging URL.

## Local API Caveat with Astro

Payload has a Local API example for Astro in their repo, but **the Local API does not work reliably with Astro's SSG mode**. The known error is `"__dirname is not defined in ES module scope"`.

**Recommendation:** Use the REST API for Astro integration. The Local API is designed for Next.js where Payload runs in the same process. For Astro, fetch from Payload's REST endpoints instead.
