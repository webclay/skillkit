---
name: payload
description: Use this skill when building with Payload CMS for content management, admin panels, or headless backend features. Activate when the user mentions Payload, CMS collections, media management, content blocks, layout builder, or admin panel customization.
---

# Payload CMS

Open-source backend framework that drops into Next.js and provides a full admin UI, REST API, authentication, and database management out of the box. Works excellently as a headless CMS with any frontend (Astro, Next.js, SvelteKit, etc.).

## What Payload Is

Payload is a config-based backend framework written in TypeScript that gives you:

- Full admin UI (auto-generated)
- Complete backend architecture
- Authentication built-in
- Auto-mounted REST API endpoints
- Database management (MongoDB, PostgreSQL, SQLite)
- File/media handling with image processing

**Key advantage:** Truly open source - no tiers, no quotas. Host anywhere (Vercel, Netlify, Railway, etc.).

**Use cases:** CMS, e-commerce, digital asset management, app backends - anything needing a backend with admin panel.

## When to Use This Skill

- Building a headless CMS
- Need an admin panel for content management
- Want a type-safe backend framework with Next.js
- Building e-commerce or digital asset management
- Need versioning, drafts, and content workflows
- Want REST APIs auto-generated from your schema
- Need file uploads with image processing
- Backend for Astro/React/Vue frontends (see `framework/astro` skill)

## When NOT to Use

- Simple static sites (use Astro or vanilla Next.js)
- No admin panel needed (use direct database access)
- Real-time collaborative editing (use dedicated realtime platforms)
- Extremely high-traffic APIs (consider specialized API frameworks)

## Installation

```bash
pnpx create-payload-app@latest
# Also works with: npx, bunx, yarn
```

The CLI guides you through:
1. Project name
2. Template selection (blank, website, etc.)
3. Database choice (MongoDB, SQLite, PostgreSQL)

After installation:
```bash
cd <project-name>
[runner] dev
```

Server runs on `localhost:3000`. Admin panel lives at `/admin`.

### Templates

- **Blank** - Minimal (users + media collections only)
- **Website** - Production-grade with live preview, revalidation, static rendering, forms, redirects, sitemap

## Core Concept: Config-Based Architecture

Everything starts with the **Payload config file** (`payload.config.ts`). This is the single source of truth for your entire backend.

Changes are picked up instantly via Next.js HMR - no restart needed.

## Collections

Collections are the primary building blocks. They map to database tables and auto-generate admin UI, REST API endpoints, and TypeScript types.

```typescript
{
  slug: 'cars',
  fields: [
    { name: 'title', type: 'text', required: true }
  ],
  admin: {
    useAsTitle: 'title',
    defaultColumns: ['title', 'status', 'updatedAt'],
  },
  versions: { drafts: true, maxPerDoc: 50 },
  access: {
    read: () => true,
    create: ({ req: { user } }) => !!user,
  },
  hooks: {
    beforeChange: [],
    afterRead: [],
  },
}
```

> Full collection options, field types, blocks, and media handling: see **collections-and-fields.md**

## Field Types Summary

| Type | Purpose |
|------|---------|
| `text` | Single-line text input |
| `number` | Numeric input |
| `select` | Dropdown with options |
| `checkbox` | Boolean toggle |
| `date` | Date picker |
| `email` | Email input with validation |
| `textarea` | Multi-line text |
| `upload` | File/image (links to media collection) |
| `relationship` | Link to another collection (single or hasMany) |
| `join` | Bidirectional relationship (reverse side) |
| `richText` | Lexical editor with extensible features |
| `blocks` | Array of typed block components (layout building) |
| `array` | Repeatable groups of fields |
| `group` | Grouped fields (nested object) |

> Full field examples and block patterns: see **collections-and-fields.md**

## Rich Text and Custom Components

Payload uses Lexical for rich text. You can add custom full-width blocks (`BlocksFeature`) and inline blocks (`InlineBlocksFeature`) inside the editor. Custom React components can replace field labels, entire fields, error displays, and descriptions in the admin UI.

> Full examples: see **rich-text-and-components.md**

## Access Control

Functional, fully typed per-operation access rules. Return `true`/`false` for simple checks, or return query constraints for filtered access.

```typescript
access: {
  read: ({ req: { user } }) => {
    if (user) return true;
    return { _status: { equals: 'published' } };
  },
  create: ({ req: { user } }) => !!user,
  delete: ({ req: { user } }) => user?.role === 'admin',
}
```

Applies automatically to Admin UI, REST API, and Local API.

> Full access control and hooks reference: see **access-hooks-versions.md**

## Hooks

Inject custom logic at lifecycle points: `beforeChange`, `beforeRead`, `beforeDelete`, `afterChange`, `afterRead`, `afterDelete`.

```typescript
hooks: {
  beforeChange: [({ data }) => { data.slug = slugify(data.title); return data; }],
  afterChange: [async ({ doc, operation }) => {
    if (operation === 'create') await sendNotification(doc);
  }],
}
```

> Full hooks reference: see **access-hooks-versions.md**

## REST API

Auto-generated endpoints for every collection:

```
GET    /api/cars          - List (supports where, limit, page, sort, depth)
GET    /api/cars/:id      - Get by ID
POST   /api/cars          - Create
PATCH  /api/cars/:id      - Update
DELETE /api/cars/:id      - Delete
```

> Query parameter examples and common patterns: see **api.md**

## Local API (Next.js Integration)

Use Payload directly in server components, route handlers, and server actions:

```typescript
import { getPayload } from 'payload';
import config from '@payload-config';

const payload = await getPayload({ config });

const cars = await payload.find({
  collection: 'cars',
  where: { featured: { equals: true } },
  select: { title: true, price: true },
  limit: 10,
});
```

Methods: `find`, `findByID`, `create`, `update`, `delete`, `auth`.

**Select API:** Pass a `select` object to return only specific fields with full TypeScript inference.

> Full Local API examples, Select API, and common patterns (blog, e-commerce): see **api.md**

## Auto-Generated Types

Payload generates TypeScript types for all collections automatically. Import from `@/payload-types`:

```typescript
import type { Car, Page } from '@/payload-types';
```

Types regenerate when you change your config.

## Versions and Drafts

Enable with `versions: { drafts: true, maxPerDoc: 50 }` and optionally `autosave: true`. Provides draft/published status, version history with rollback, and compare.

> Full details: see **access-hooks-versions.md**

## Form Builder Plugin

The `@payloadcms/plugin-form-builder` adds complete form management - create forms in admin, manage submissions, render on frontend.

> Full setup, configuration, and rendering: see **form-builder.md**

## Headless CMS Pattern (Payload + Astro)

Use Payload as a backend with any frontend framework. Recommended architecture: monorepo with Payload backend and Astro (or other) frontend, communicating via REST API.

> Full architecture, setup, fetching, block rendering, deployment options, build triggers, live preview: see **headless-cms.md**

## Deploying on Cloudflare Workers

Official support for Cloudflare Workers with D1 (SQLite) and R2 (storage). Recommended for full Cloudflare stack (~$5/month). Uses `@payloadcms/db-d1-sqlite`, `@payloadcms/storage-r2`, and OpenNext.

Alternative: Postgres + Hyperdrive for connection pooling on Workers.

> Full Cloudflare deployment guide, Wrangler config, read replicas, staging, cost estimates: see **deployment.md**

## Runtime Compatibility (Bun / Node.js)

Payload does NOT officially support Bun as a runtime. Use Bun as package manager, Node.js as runtime for Payload CLI commands. The `--disable-transpile` flag is the workaround for Bun runtime.

> Full compatibility details: see **deployment.md**

## Tips and Best Practices

1. **Start with the blank template** - Build up from minimal config
2. **Use relationships over duplication** - Keep data normalized
3. **Use join fields** for bidirectional relationships - Better admin UX
4. **Leverage Select API** - Only fetch fields you need
5. **Use hooks for computed fields** - Keep logic in one place
6. **Version control important content** - Pages, posts, products
7. **Set up access control early** - Think about permissions upfront
8. **Use auto-generated types** - Let Payload handle TypeScript
9. **Custom React components** for better UX - Make admin friendly for editors
10. **Image sizes in media collection** - Define all sizes upfront

## Auto-Seeding Admin Users

Content website templates can include an `onInit` hook in `payload.config.ts` that automatically creates admin accounts on first startup. This removes the manual "create first admin" step.

**How it works:**
1. On every Payload startup, the `onInit` hook checks if the `users` collection is empty
2. If no users exist (first run), it reads `PAYLOAD_ADMIN_EMAILS` (comma-separated) and creates an admin account for each email
3. All accounts use the password from `PAYLOAD_ADMIN_PASSWORD`
4. On subsequent startups, the hook finds existing users and skips seeding entirely

**Environment variables:**

| Variable | Purpose | Default |
|----------|---------|---------|
| `PAYLOAD_ADMIN_EMAILS` | Comma-separated list of admin emails to create | (empty - no seeding) |
| `PAYLOAD_ADMIN_PASSWORD` | Password for all seeded accounts | `changeme123` |

**Where values come from:** The admin seeding config (email templates, default password) is user-specific and stored in each project's `dev-context.md` under `## Admin Seeding`. The setup wizard reads this config and writes the resolved emails into `.env`. Email templates can use `{project}` as a placeholder for the lowercase project folder name.

**Important notes:**
- Seeding only happens when the users collection is completely empty - it's safe to restart Payload
- If `PAYLOAD_ADMIN_EMAILS` is empty, no seeding occurs and Payload shows the manual registration form
- In production, change `PAYLOAD_ADMIN_PASSWORD` to a strong password via the deployment platform's environment variables
- The seeded accounts have full admin access to all collections
- See [headless-cms.md](headless-cms.md) for the full `payload.config.ts` template with the `onInit` hook

## How to Verify

### Quick Checks
- `[runner] dev` starts without errors
- Admin panel loads at `/admin`
- Collections appear in admin UI
- Can create/read/update/delete documents
- API endpoints respond at `/api/<collection-slug>`
- Types are generated in `payload-types.ts`

### Common Issues
- **"Cannot find module '@payload-config'"**: Check import path matches your config file location
- **Types not updating**: Restart dev server or run `payload generate:types`
- **Images not uploading**: Check `upload` directory permissions and config
- **Access denied in admin**: Check `access` rules on collections
- **Hooks not firing**: Verify hook array syntax and placement

## Next Steps

After basic setup:
1. Define your collections and fields
2. Set up relationships between collections
3. Configure media handling and image sizes
4. Add access control rules
5. Implement hooks for business logic
6. Customize rich text with blocks
7. Build your frontend using the Local API
8. Deploy (Vercel, Netlify, Railway, etc.)

Payload handles the backend - you focus on the data model and frontend.

## Reference Files

| File | Contents |
|------|----------|
| [collections-and-fields.md](collections-and-fields.md) | Full collection options, all field types with examples, blocks (defining, using, rendering), media collection with image processing |
| [rich-text-and-components.md](rich-text-and-components.md) | Lexical rich text customization (custom blocks, inline blocks), custom React admin components |
| [access-hooks-versions.md](access-hooks-versions.md) | Access control (basic + query constraints), lifecycle hooks, auto-generated types, versions and drafts |
| [api.md](api.md) | REST API endpoints and query params, Local API methods, Select API, common patterns (blog, e-commerce) |
| [form-builder.md](form-builder.md) | Form Builder plugin setup, admin usage, frontend rendering, viewing submissions |
| [headless-cms.md](headless-cms.md) | Headless CMS architecture (Payload + Astro), monorepo setup, fetching, block rendering, deployment options, build triggers, live preview |
| [deployment.md](deployment.md) | Cloudflare Workers deployment (D1 + R2), Wrangler config, read replicas, Postgres + Hyperdrive, staging, costs, Bun/Node.js runtime compatibility |
