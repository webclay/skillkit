---
name: payload
description: Payload CMS - Open-source TypeScript backend framework with built-in admin panel. Trigger words - Payload, CMS, headless CMS, admin panel, content management, backend framework, collections, media management, blocks, layout builder
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

Collections are the primary building blocks. They map to database tables and auto-generate:
- Admin UI
- REST API endpoints
- TypeScript types

### Defining a Collection

```typescript
// In payload.config.ts
{
  slug: 'cars',
  fields: [
    {
      name: 'title',
      type: 'text',
      required: true,
    }
  ],
  admin: {
    useAsTitle: 'title', // Which field shows as document title in admin
  }
}
```

### Key Collection Options

```typescript
{
  slug: 'pages',
  fields: [...],

  // Admin UI config
  admin: {
    useAsTitle: 'title',
    defaultColumns: ['title', 'status', 'updatedAt'],
  },

  // Versioning and drafts
  versions: {
    drafts: true,
    maxPerDoc: 50,
  },

  // Auto-save drafts
  autosave: true,

  // Access control (see Access Control section)
  access: {
    read: () => true,
    create: ({ req: { user } }) => !!user,
  },

  // Lifecycle hooks (see Hooks section)
  hooks: {
    beforeChange: [],
    afterRead: [],
  },
}
```

## Field Types

All fields require at minimum `name` and `type`.

### Common Field Types

```typescript
// Text input
{
  name: 'title',
  type: 'text',
  required: true,
}

// Number
{
  name: 'price',
  type: 'number',
  min: 0,
}

// Select dropdown
{
  name: 'status',
  type: 'select',
  options: ['draft', 'published', 'archived'],
  defaultValue: 'draft',
}

// Checkbox
{
  name: 'featured',
  type: 'checkbox',
  defaultValue: false,
}

// Date
{
  name: 'publishDate',
  type: 'date',
}

// Email
{
  name: 'email',
  type: 'email',
  required: true,
}

// Textarea
{
  name: 'description',
  type: 'textarea',
}
```

### Upload (File/Image)

```typescript
{
  name: 'featuredImage',
  type: 'upload',
  relationTo: 'media', // Points to the media collection
  required: true,
}
```

### Relationship

```typescript
{
  name: 'manufacturer',
  type: 'relationship',
  relationTo: 'manufacturers',
  required: true,
}

// Multiple relationships
{
  name: 'categories',
  type: 'relationship',
  relationTo: 'categories',
  hasMany: true, // Allows multiple selections
}

// Polymorphic relationships
{
  name: 'relatedItems',
  type: 'relationship',
  relationTo: ['cars', 'manufacturers'], // Can relate to multiple collections
  hasMany: true,
}
```

### Join (Bidirectional Relationships)

Create automatic links from the "other side" of a relationship.

```typescript
// On the manufacturers collection
{
  name: 'cars',
  type: 'join',
  collection: 'cars', // Which collection to join from
  on: 'manufacturer',  // Which field on that collection points here
}
```

This shows all cars related to a manufacturer in the admin UI with direct navigation.

### Rich Text (Lexical Editor)

```typescript
{
  name: 'content',
  type: 'richText',
}
```

The Lexical editor is highly extensible (see Rich Text Customization section).

### Blocks (Layout Building)

**Blocks are the key to building flexible, modular pages.** Think of blocks as Legos - editors can add, remove, and rearrange them to build pages.

**Core concept:** A `blocks` field contains an array of different block types. Each block type defines its own set of fields. Editors select which blocks to add and in what order.

**Common use case:** Pages collection with a `layout` field containing hero blocks, content blocks, form blocks, etc.

```typescript
{
  name: 'layout',
  type: 'blocks',
  blocks: [
    // Each block defined here becomes available in the admin UI
    HeroBlock,
    ContentBlock,
    FormBlock,
  ],
}
```

#### Defining Blocks

Blocks are defined separately and imported into your collection. Best practice: create a `blocks/` directory to organize them.

**Hero Block Example:**

```typescript
// src/blocks/HeroBlock.ts
import { Block } from 'payload';

export const HeroBlock: Block = {
  slug: 'hero',
  fields: [
    {
      name: 'heading',
      type: 'text',
      required: true,
    },
    {
      name: 'subheading',
      type: 'richText',
    },
    {
      name: 'image',
      type: 'upload',
      relationTo: 'media',
    },
    {
      name: 'ctaButton',
      type: 'group',
      fields: [
        {
          name: 'label',
          type: 'text',
          required: true,
        },
        {
          name: 'url',
          type: 'text',
          required: true,
        },
      ],
    },
  ],
};
```

**Content Block Example:**

```typescript
// src/blocks/ContentBlock.ts
import { Block } from 'payload';

export const ContentBlock: Block = {
  slug: 'content',
  fields: [
    {
      name: 'heading',
      type: 'text',
    },
    {
      name: 'content',
      type: 'richText',
      required: true,
    },
  ],
};
```

**Form Block Example:**

```typescript
// src/blocks/FormBlock.ts
import { Block } from 'payload';

export const FormBlock: Block = {
  slug: 'form',
  fields: [
    {
      name: 'heading',
      type: 'text',
    },
    {
      name: 'form',
      type: 'relationship',
      relationTo: 'forms',
      required: true,
    },
  ],
};
```

#### Using Blocks in Collections

```typescript
// collections/Pages.ts
import { HeroBlock } from '../blocks/HeroBlock';
import { ContentBlock } from '../blocks/ContentBlock';
import { FormBlock } from '../blocks/FormBlock';

export const Pages: CollectionConfig = {
  slug: 'pages',
  fields: [
    {
      name: 'title',
      type: 'text',
      required: true,
    },
    {
      name: 'slug',
      type: 'text',
      required: true,
      unique: true,
    },
    {
      name: 'layout',
      type: 'blocks',
      blocks: [HeroBlock, ContentBlock, FormBlock],
    },
  ],
  admin: {
    useAsTitle: 'title',
  },
};
```

#### Rendering Blocks on the Frontend

When you fetch a page, the `layout` field contains an array of blocks with a `blockType` property.

**Pattern: Switch statement to render different block types**

```typescript
// Render blocks helper
function renderBlocks(blocks: any[]) {
  return blocks.map((block, index) => {
    switch (block.blockType) {
      case 'hero':
        return <HeroBlock key={block.id || index} block={block} />;
      case 'content':
        return <ContentBlock key={block.id || index} block={block} />;
      case 'form':
        return <FormBlock key={block.id || index} block={block} />;
      default:
        return null;
    }
  });
}

// In your page component
export default async function Page({ params }: { params: { slug: string } }) {
  const payload = await getPayload({ config });

  const page = await payload.find({
    collection: 'pages',
    where: { slug: { equals: params.slug } },
  });

  if (!page.docs[0]) {
    return <div>Page not found</div>;
  }

  const { layout } = page.docs[0];

  return <main>{renderBlocks(layout)}</main>;
}
```

**Hero Block Component Example:**

```typescript
// components/HeroBlock.tsx
import { RichText } from '@payloadcms/richtext-lexical/react';
import Image from 'next/image';

interface HeroBlockProps {
  block: {
    heading: string;
    subheading: any; // Rich text
    image: {
      url: string;
      alt: string;
      width: number;
      height: number;
    };
    ctaButton: {
      label: string;
      url: string;
    };
  };
}

export function HeroBlock({ block }: HeroBlockProps) {
  return (
    <section className="hero">
      <h1>{block.heading}</h1>
      <RichText data={block.subheading} />
      {block.image && (
        <Image
          src={block.image.url}
          alt={block.image.alt}
          width={block.image.width}
          height={block.image.height}
        />
      )}
      {block.ctaButton && (
        <a href={block.ctaButton.url} className="cta-button">
          {block.ctaButton.label}
        </a>
      )}
    </section>
  );
}
```

**Content Block Component Example:**

```typescript
// components/ContentBlock.tsx
import { RichText } from '@payloadcms/richtext-lexical/react';

interface ContentBlockProps {
  block: {
    heading?: string;
    content: any; // Rich text
  };
}

export function ContentBlock({ block }: ContentBlockProps) {
  return (
    <section className="content">
      {block.heading && <h2>{block.heading}</h2>}
      <RichText data={block.content} />
    </section>
  );
}
```

#### Block Organization Best Practices

1. **One block per file** - Keep blocks in `src/blocks/` directory
2. **Descriptive slugs** - Use clear, semantic slugs (`hero`, `content`, `testimonials`)
3. **Reusable blocks** - Design blocks to work across multiple collections
4. **Type safety** - Extract types from Payload's auto-generated types

```typescript
import type { Page } from '@/payload-types';

type HeroBlock = Extract<Page['layout'][0], { blockType: 'hero' }>;
```

## Media Collection

The media collection handles file uploads and includes powerful image processing.

### Image Sizes and Processing

```typescript
{
  slug: 'media',
  upload: {
    // Define additional image sizes - automatically generated on upload
    imageSizes: [
      {
        name: 'thumbnail',
        width: 300,
        height: 300,
        crop: 'center', // or 'top', 'bottom', 'left', 'right', 'focalpoint'
      },
      {
        name: 'banner',
        width: 1200,
        height: 400,
        crop: 'focalpoint', // Uses focal point from admin UI
      },
      {
        name: 'hero',
        width: 1920,
        height: 1080,
      }
    ],

    // Crop controls in admin UI
    cropControls: true,

    // Focal point controls
    focalPoint: true,
  },
  fields: [
    {
      name: 'alt',
      type: 'text',
      required: true,
    }
  ]
}
```

**Features:**
- Automatic image resizing
- Built-in crop UI
- Hotspot/focal point selection
- Multiple sizes generated per upload

## Rich Text Customization

Payload uses Lexical and supports custom blocks within rich text.

### Custom Blocks (Full-Width)

```typescript
// Define the block
const CarHighlight = {
  slug: 'carHighlight',
  fields: [
    {
      name: 'car',
      type: 'relationship',
      relationTo: 'cars',
      required: true,
    },
    {
      name: 'highlightType',
      type: 'select',
      options: ['image', 'gallery', 'specs'],
      defaultValue: 'image',
    }
  ],
};

// Add to rich text field
{
  name: 'content',
  type: 'richText',
  editor: lexicalEditor({
    features: ({ defaultFeatures }) => [
      ...defaultFeatures,
      BlocksFeature({
        blocks: [CarHighlight],
      }),
    ],
  }),
}
```

### Inline Blocks

```typescript
// Inline block for dynamic references
const CarReference = {
  slug: 'carReference',
  fields: [
    {
      name: 'car',
      type: 'relationship',
      relationTo: 'cars',
      required: true,
    },
    {
      name: 'displayField',
      type: 'select',
      options: ['title', 'price', 'year'],
    }
  ],
};

// Add to rich text as inline
{
  name: 'content',
  type: 'richText',
  editor: lexicalEditor({
    features: ({ defaultFeatures }) => [
      ...defaultFeatures,
      InlineBlocksFeature({
        inlineBlocks: [CarReference],
      }),
    ],
  }),
}
```

Editors can insert these blocks directly in the text flow, and values populate dynamically on the frontend.

## Custom React Components

Inject custom React components anywhere in the admin UI.

```typescript
// Custom label component
const CustomLabel: React.FC = () => {
  const { value: carId } = useField<string>({ path: 'car' });
  const [carTitle, setCarTitle] = useState('');

  useEffect(() => {
    if (carId) {
      fetch(`/api/cars/${carId}`)
        .then(r => r.json())
        .then(data => setCarTitle(data.title));
    }
  }, [carId]);

  return <span>{carTitle || 'Select a car'}</span>;
};

// Use in field definition
{
  name: 'car',
  type: 'relationship',
  relationTo: 'cars',
  admin: {
    components: {
      Label: CustomLabel,
    }
  }
}
```

You can customize:
- `Label` - Field label
- `Field` - Entire field component
- `Error` - Error message display
- `Description` - Field description

## Access Control

Access control is functional and fully typed. Define per-operation access rules.

### Basic Access Control

```typescript
{
  slug: 'pages',
  access: {
    // Anyone can read
    read: () => true,

    // Only logged-in users can create
    create: ({ req: { user } }) => !!user,

    // Only admins can delete
    delete: ({ req: { user } }) => {
      return user?.role === 'admin';
    },
  }
}
```

### Query Constraints

Return query constraints to filter data based on user context.

```typescript
{
  slug: 'pages',
  access: {
    read: ({ req: { user } }) => {
      // Logged-in users see everything
      if (user) return true;

      // Anonymous users only see published docs
      return {
        _status: { equals: 'published' }
      };
    },

    // Users can only edit their own pages
    update: ({ req: { user } }) => {
      if (user?.role === 'admin') return true;

      return {
        author: { equals: user?.id }
      };
    },
  }
}
```

Access control applies automatically to:
- Admin UI
- REST API
- Local API

## Hooks

Inject custom logic at specific points in the CRUD lifecycle.

### Available Hooks

```typescript
{
  slug: 'cars',
  hooks: {
    // Before operations
    beforeChange: [
      ({ data, req }) => {
        // Modify data before save
        data.slug = slugify(data.title);
        return data;
      }
    ],
    beforeRead: [({ doc, req }) => { /* ... */ }],
    beforeDelete: [({ req, id }) => { /* ... */ }],

    // After operations
    afterChange: [
      async ({ doc, req, operation }) => {
        // Send notification, revalidate cache, etc.
        if (operation === 'create') {
          await sendNotification(doc);
        }
      }
    ],
    afterRead: [
      ({ doc }) => {
        // Add computed properties
        doc.isVintage = doc.year < 1980;
        return doc;
      }
    ],
    afterDelete: [({ doc, req }) => { /* ... */ }],
  }
}
```

### Hook Parameters

All hooks receive:
- `req` - Request object with `user`, `payload`, etc.
- `doc` - The document (in afterRead, afterChange, etc.)
- `data` - The incoming data (in beforeChange)
- `operation` - The operation type ('create', 'update', etc.)

## REST API

Payload auto-generates REST endpoints for every collection.

### Auto-Generated Endpoints

```
GET    /api/cars          - List all cars
GET    /api/cars/:id      - Get specific car
POST   /api/cars          - Create car
PATCH  /api/cars/:id      - Update car
DELETE /api/cars/:id      - Delete car
```

### Query Parameters

```
GET /api/cars?where[manufacturer][equals]=ferrari
GET /api/cars?limit=10&page=2
GET /api/cars?sort=-createdAt
GET /api/cars?depth=2  // Populate relationships
```

Access control applies automatically to all API requests.

## Local API (Next.js Integration)

Use Payload directly in Next.js route handlers, server components, and server actions.

### In Route Handlers

```typescript
// app/my-route/route.ts
import { getPayload } from 'payload';
import config from '@payload-config';

export async function GET(req: Request) {
  const payload = await getPayload({ config });

  // Check authentication
  const { user } = await payload.auth({ headers: req.headers });

  if (!user) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 });
  }

  // Query with Select API
  const cars = await payload.find({
    collection: 'cars',
    select: {
      title: true,
      price: true,
      manufacturer: true,
    },
    where: {
      featured: { equals: true }
    },
    limit: 10,
  });

  return Response.json({ cars: cars.docs });
}
```

### In Server Components

```typescript
// app/cars/page.tsx
import { getPayload } from 'payload';
import config from '@payload-config';

export default async function CarsPage() {
  const payload = await getPayload({ config });

  const cars = await payload.find({
    collection: 'cars',
    where: { status: { equals: 'published' } },
  });

  return (
    <div>
      {cars.docs.map(car => (
        <CarCard key={car.id} car={car} />
      ))}
    </div>
  );
}
```

### Local API Methods

```typescript
// Find
const result = await payload.find({
  collection: 'cars',
  where: { ... },
  limit: 10,
  page: 1,
  sort: '-createdAt',
});

// Find by ID
const car = await payload.findByID({
  collection: 'cars',
  id: '123',
});

// Create
const newCar = await payload.create({
  collection: 'cars',
  data: {
    title: 'New Car',
    price: 50000,
  },
});

// Update
const updated = await payload.update({
  collection: 'cars',
  id: '123',
  data: {
    price: 45000,
  },
});

// Delete
await payload.delete({
  collection: 'cars',
  id: '123',
});

// Authentication
const { user } = await payload.auth({ headers });
```

## Select API

Choose specific fields to return - keeps responses lean and typed.

```typescript
const cars = await payload.find({
  collection: 'cars',
  select: {
    title: true,
    price: true,
    manufacturer: {
      name: true,
      country: true,
    },
  },
});

// TypeScript knows exact shape:
// { title: string; price: number; manufacturer: { name: string; country: string } }
```

Only requested fields are returned and typed accordingly.

## Auto-Generated Types

Payload generates TypeScript types for all collections.

```typescript
import type { Car, Manufacturer, Page } from '@/payload-types';

// Fully typed based on your collection schemas
const car: Car = await payload.findByID({
  collection: 'cars',
  id: '123',
});

// TypeScript knows all fields, relationships, etc.
console.log(car.title); // ✓
console.log(car.manufacturer.name); // ✓
```

Types regenerate automatically when you change your config.

## Versions and Drafts

Enable version control and draft/publish workflows.

```typescript
{
  slug: 'pages',
  versions: {
    drafts: true,      // Enable draft/published status
    maxPerDoc: 50,     // Keep last 50 versions
  },
  autosave: true,      // Auto-save drafts while editing
}
```

**Features:**
- Draft vs. published status
- Version history with rollback
- Save changes without publishing
- Compare versions

Access control can filter by `_status`:

```typescript
access: {
  read: ({ req: { user } }) => {
    if (user) return true;
    return { _status: { equals: 'published' } };
  }
}
```

## Common Patterns

### Blog with Categories and Authors

```typescript
const collections = [
  {
    slug: 'posts',
    fields: [
      { name: 'title', type: 'text', required: true },
      { name: 'slug', type: 'text', required: true, unique: true },
      { name: 'content', type: 'richText' },
      {
        name: 'author',
        type: 'relationship',
        relationTo: 'users',
        required: true,
      },
      {
        name: 'categories',
        type: 'relationship',
        relationTo: 'categories',
        hasMany: true,
      },
      {
        name: 'featuredImage',
        type: 'upload',
        relationTo: 'media',
      },
      { name: 'publishedAt', type: 'date' },
    ],
    admin: { useAsTitle: 'title' },
    versions: { drafts: true },
  },
  {
    slug: 'categories',
    fields: [
      { name: 'name', type: 'text', required: true },
      { name: 'slug', type: 'text', required: true },
    ],
    admin: { useAsTitle: 'name' },
  },
];
```

### E-commerce Products

```typescript
{
  slug: 'products',
  fields: [
    { name: 'name', type: 'text', required: true },
    { name: 'description', type: 'richText' },
    { name: 'price', type: 'number', required: true, min: 0 },
    {
      name: 'images',
      type: 'upload',
      relationTo: 'media',
      hasMany: true,
    },
    {
      name: 'category',
      type: 'relationship',
      relationTo: 'categories',
      required: true,
    },
    {
      name: 'variants',
      type: 'array',
      fields: [
        { name: 'name', type: 'text' },
        { name: 'price', type: 'number' },
        { name: 'sku', type: 'text' },
        { name: 'stock', type: 'number', min: 0 },
      ],
    },
    { name: 'featured', type: 'checkbox', defaultValue: false },
  ],
  admin: { useAsTitle: 'name' },
}
```

## Form Builder Plugin

The Form Builder plugin adds a complete form management system to Payload - create forms in the admin, manage submissions, and render forms on your frontend.

### Installation

```bash
npm install @payloadcms/plugin-form-builder
```

### Configuration

```typescript
// payload.config.ts
import { formBuilderPlugin } from '@payloadcms/plugin-form-builder';

export default buildConfig({
  plugins: [
    formBuilderPlugin({
      fields: {
        // Use default fields (text, email, textarea, select, checkbox, etc.)
      },
      // Optional: customize form submissions collection
      formSubmissionOverrides: {
        slug: 'contact-submissions',
      },
    }),
  ],
});
```

This creates two collections automatically:
- **forms** - Where you create and manage forms
- **form-submissions** - Where form submissions are stored

### Creating Forms in Admin

1. Go to **Forms** collection in admin
2. Click **Create New**
3. Add a title (e.g., "Newsletter Signup")
4. Add fields (text, email, textarea, select, checkbox, etc.)
5. Configure submit button label
6. Add confirmation message (rich text)
7. Save

**Example form fields:**
- First Name (text, required)
- Last Name (text, required)
- Email (email, required)
- Submit button label: "Sign Up"
- Confirmation: "Thanks for signing up!"

### Using Forms in Blocks

**Form Block Definition:**

```typescript
// blocks/FormBlock.ts
import { Block } from 'payload';

export const FormBlock: Block = {
  slug: 'form',
  fields: [
    {
      name: 'heading',
      type: 'text',
    },
    {
      name: 'form',
      type: 'relationship',
      relationTo: 'forms',
      required: true,
    },
  ],
};
```

Add to your Pages collection's layout blocks, and editors can select which form to display.

### Rendering Forms on Frontend

```typescript
// components/FormBlock.tsx
'use client';

import { useState, FormEvent } from 'react';
import { RichText } from '@payloadcms/richtext-lexical/react';

interface FormBlockProps {
  block: {
    heading?: string;
    form: {
      id: string;
      title: string;
      fields: Array<{
        name: string;
        label: string;
        fieldType: string;
        required: boolean;
      }>;
      submitButtonLabel: string;
      confirmationMessage: any; // Rich text
    };
  };
}

export function FormBlock({ block }: FormBlockProps) {
  const { heading, form } = block;
  const [formState, setFormState] = useState({
    loading: false,
    error: null,
    success: false,
  });

  const handleSubmit = async (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setFormState({ loading: true, error: null, success: false });

    const formData = new FormData(e.currentTarget);
    const data = Object.fromEntries(formData.entries());

    try {
      // Submit to Payload's form submission endpoint
      const response = await fetch('/api/form-submissions', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          form: form.id,
          submissionData: Object.entries(data).map(([field, value]) => ({
            field,
            value: value as string,
          })),
        }),
      });

      if (!response.ok) throw new Error('Failed to submit form');

      setFormState({ loading: false, error: null, success: true });
      e.currentTarget.reset();

      // Clear success message after 5 seconds
      setTimeout(() => {
        setFormState({ loading: false, error: null, success: false });
      }, 5000);
    } catch (error) {
      setFormState({
        loading: false,
        error: 'Failed to submit form',
        success: false,
      });
    }
  };

  return (
    <section className="form-block">
      {heading && <h2>{heading}</h2>}

      <form onSubmit={handleSubmit}>
        {form.fields.map((field) => (
          <div key={field.name} className="form-field">
            <label htmlFor={field.name}>{field.label}</label>
            <input
              id={field.name}
              name={field.name}
              type={field.fieldType}
              required={field.required}
              placeholder={field.label}
            />
          </div>
        ))}

        {formState.error && <p className="error">{formState.error}</p>}

        {formState.success ? (
          <div className="success">
            <RichText data={form.confirmationMessage} />
          </div>
        ) : (
          <button type="submit" disabled={formState.loading}>
            {formState.loading ? 'Submitting...' : form.submitButtonLabel}
          </button>
        )}
      </form>
    </section>
  );
}
```

### Viewing Form Submissions

All submissions appear in the **Form Submissions** collection in the admin panel, organized by form. Each submission shows:
- Which form it came from
- Submitted data
- Submission timestamp
- IP address (optional)

## Headless CMS Pattern (Payload + Astro)

Payload works excellently as a headless CMS - use Payload for the backend/admin and any frontend framework (Astro, Next.js, SvelteKit, etc.) to render the content.

**See also:** The `framework/astro` skill has complete patterns for using Astro as a frontend with Payload CMS backend.

### Architecture

**Payload Backend:**
- Collections, fields, relationships
- Admin panel at `/admin`
- REST API at `/api/*`
- Hosted separately or on the same server

**Astro Frontend:**
- Fetches content from Payload REST API
- Renders blocks as Astro components
- Fast, static-first builds

### Setup Pattern

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

### Monorepo Development Workflow

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

### Fetching Content from Payload in Astro

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

### Rendering Blocks in Astro

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

### Deployment

**Option 1: Full Cloudflare Stack (Recommended)**
- Payload on Cloudflare Workers (D1 database + R2 storage)
- Astro on Cloudflare Pages
- Single platform, ~$5/month total
- See "Deploying Payload on Cloudflare Workers" section below

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

### Build Trigger Strategies

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

### Live Preview with Astro

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

### Local API Caveat with Astro

Payload has a Local API example for Astro in their repo, but **the Local API does not work reliably with Astro's SSG mode**. The known error is `"__dirname is not defined in ES module scope"`.

**Recommendation:** Use the REST API for Astro integration. The Local API is designed for Next.js where Payload runs in the same process. For Astro, fetch from Payload's REST endpoints instead.

## Deploying Payload on Cloudflare Workers

Payload has official support for Cloudflare Workers. This is the recommended deployment when using Astro on Cloudflare Pages - keep the entire stack on Cloudflare.

**What you get:**
- **Cloudflare Workers** - Serverless platform running Payload (paid plan, ~$5/month)
- **D1** - SQLite database with global read replicas (reads from nearest node)
- **R2** - Object storage for media/uploads
- **Wrangler** - CLI for local dev and deployment
- **OpenNext** - Converts Next.js (which Payload runs on) to deploy on Workers

### Quick Start (One-Click Template)

Payload provides an official one-click deploy template at `payloadcms/payload/templates/with-cloudflare-d1`. It creates a D1 database and R2 bucket automatically bound to your worker.

For manual setup, start from the template and customize.

### Required Packages

```bash
pnpm add @payloadcms/db-d1-sqlite @payloadcms/storage-r2 @opennextjs/cloudflare cross-env
```

### Wrangler Configuration

```jsonc
// wrangler.jsonc
{
  "$schema": "node_modules/wrangler/config-schema.json",
  "main": ".open-next/worker.js",
  "name": "my-app",
  "compatibility_date": "2025-08-15",
  "compatibility_flags": [
    "nodejs_compat",
    "global_fetch_strictly_public"
  ],
  "assets": {
    "directory": ".open-next/assets",
    "binding": "ASSETS"
  },
  "d1_databases": [{
    "binding": "D1",
    "database_id": "<your D1 database ID>",
    "database_name": "my-app",
    "remote": true
  }],
  "r2_buckets": [{
    "binding": "R2",
    "bucket_name": "my-app"
  }]
}
```

**Important:** Bindings are `D1` and `R2` (not connection strings). PAYLOAD_SECRET is set as a Cloudflare secret, not in wrangler.jsonc.

### Payload Config for Cloudflare

The config needs to resolve bindings differently for local dev (via Wrangler proxy) vs production (via OpenNext context):

```typescript
// src/payload.config.ts
import { sqliteD1Adapter } from '@payloadcms/db-d1-sqlite';
import { r2Storage } from '@payloadcms/storage-r2';
import { lexicalEditor } from '@payloadcms/richtext-lexical';
import { buildConfig } from 'payload';
import { getCloudflareContext } from '@opennextjs/cloudflare';

const isProduction = process.env.NODE_ENV === 'production';

// Resolve Cloudflare bindings (D1, R2) for both local and production
const cloudflare = isProduction
  ? await getCloudflareContext({ async: true })
  : await import('wrangler').then(({ getPlatformProxy }) =>
      getPlatformProxy({ environment: process.env.CLOUDFLARE_ENV })
    );

export default buildConfig({
  editor: lexicalEditor(),
  secret: process.env.PAYLOAD_SECRET || '',
  collections: [Users, Media], // Your collections
  db: sqliteD1Adapter({
    binding: cloudflare.env.D1, // Passes actual D1 binding object
    // readReplicas: 'first-primary', // Enable after turning on in CF dashboard
  }),
  plugins: [
    r2Storage({
      bucket: cloudflare.env.R2, // Passes actual R2 binding object
      collections: { media: true },
    }),
  ],
});
```

**Key detail:** The adapter receives the actual binding objects (`cloudflare.env.D1`, `cloudflare.env.R2`), not string names. Wrangler automatically creates local proxies for these during development.

### Package.json Scripts

```json
{
  "scripts": {
    "dev": "cross-env NODE_OPTIONS=--no-deprecation next dev",
    "build": "cross-env NODE_OPTIONS=\"--no-deprecation --max-old-space-size=8000\" next build",
    "deploy": "pnpm run deploy:database && pnpm run deploy:app",
    "deploy:app": "opennextjs-cloudflare build && opennextjs-cloudflare deploy",
    "deploy:database": "cross-env NODE_ENV=production PAYLOAD_SECRET=ignore payload migrate",
    "preview": "opennextjs-cloudflare build && opennextjs-cloudflare preview",
    "generate:types": "payload generate:types",
    "generate:types:cloudflare": "wrangler types --env-interface CloudflareEnv cloudflare-env.d.ts"
  }
}
```

### Local Development

```bash
# Authenticate with Cloudflare first
pnpm wrangler login

# Standard local dev - Wrangler auto-binds D1/R2 locally
pnpm dev

# Preview full Cloudflare build locally
pnpm preview
```

Wrangler automatically creates local proxies for D1 and R2 during `pnpm dev` - no manual setup needed.

### Deploy

```bash
# Create migration (after schema changes)
pnpm payload migrate:create

# Deploy database + app
pnpm run deploy
```

The deploy command runs migrations against the remote D1 database, then builds via OpenNext and deploys to Workers.

### Read Replicas

D1 supports global read replicas for faster reads from any region. Enable in two steps:

1. Add to your DB adapter config:
```typescript
db: sqliteD1Adapter({
  binding: cloudflare.env.D1,
  readReplicas: 'first-primary', // First query hits primary, subsequent reads hit nearest replica
}),
```

2. Enable read replicas in your D1 Cloudflare dashboard.

Writes always go to the primary region. Reads serve from the nearest replica.

**Real-world performance gains** (from Cloudflare's benchmarks with database in Eastern North America):

| Metric | Without replicas | With replicas | Improvement |
|--------|-----------------|---------------|-------------|
| P50 | 300ms | 120ms | -60% |
| P90 | 480ms | 250ms | -48% |
| P99 | 760ms | 550ms | -28% |

### Alternative: Postgres + Hyperdrive

If you prefer PostgreSQL over D1/SQLite, you can use Payload's official `@payloadcms/db-postgres` adapter with Cloudflare Hyperdrive for connection pooling:

```typescript
import { postgresAdapter } from '@payloadcms/db-postgres';
import { getCloudflareContext } from '@opennextjs/cloudflare';

const cloudflare = await getCloudflareContext({ async: true });

export default buildConfig({
  db: postgresAdapter({
    pool: {
      connectionString: cloudflare.env.HYPERDRIVE.connectionString,
      maxUses: 1, // Required - connections can't be shared across requests on Workers
    },
  }),
});
```

Hyperdrive maintains a connection pool across the Cloudflare network and adds a query cache. Configure it in your Cloudflare dashboard pointing to any Postgres provider (Neon, Supabase, etc.).

### Staging Environments

The template supports multiple environments via `CLOUDFLARE_ENV`:

```bash
# Deploy to staging
CLOUDFLARE_ENV=staging pnpm run deploy
```

Configure staging bindings in wrangler.jsonc under the `env` key with separate D1 databases and R2 buckets.

### Full Cloudflare Stack (Recommended for Astro + Payload)

When using both Astro and Payload on Cloudflare:

```
my-website/
├── cms/                    # Payload on Cloudflare Workers
│   ├── src/
│   │   ├── collections/
│   │   ├── blocks/
│   │   └── payload.config.ts
│   ├── wrangler.jsonc
│   └── package.json
│
├── web/                    # Astro on Cloudflare Pages
│   ├── src/
│   │   ├── pages/
│   │   ├── components/
│   │   └── lib/
│   └── package.json
```

**Environment variables:**

```env
# cms/.env (local)
PAYLOAD_SECRET=your-local-secret

# web/.env (local)
PAYLOAD_API_URL=http://localhost:3000/api
```

**Production (set in Cloudflare dashboard):**

Workers (Payload):
- `PAYLOAD_SECRET` - Secure random string (set via `wrangler secret put PAYLOAD_SECRET`)
- D1 and R2 bindings configured in wrangler.jsonc

Pages (Astro):
- `PAYLOAD_API_URL` - Your Payload Worker URL (e.g., `https://my-app.workers.dev/api`)

### Limitations

- **Paid Workers plan required** (~$5/month) due to bundle size exceeding free tier 3MB limit
- **GraphQL not fully supported** on Workers - use REST API (upstream workerd issue)
- **D1 access via bindings only** - no traditional connection strings
- **Logs disabled by default** - enable in Cloudflare dashboard (counts against quota)

### Cost Estimate (Cloudflare Stack)

| Service | Cost |
|---------|------|
| Workers (paid plan) | ~$5/month |
| D1 database | Free tier covers most CMS usage |
| R2 storage | Free for first 10GB |
| Pages (Astro) | Free |
| **Total** | **~$5/month** |

Significantly cheaper than Railway + MongoDB Atlas (~$28/month), with the bonus of global edge performance from read replicas.

## Runtime Compatibility (Bun / Node.js)

**Payload does NOT officially support Bun as a runtime.** Their test suite only runs against Node.js.

### What works with Bun

- **Bun as package manager** (`bun install`, `bun add`) - works fine
- **`bun run dev`** - works because Next.js uses Node.js under the hood
- **Payload CLI with `--disable-transpile`** - workaround for Bun runtime

### What breaks with Bun

- **Payload CLI commands under Bun runtime** - fails because Bun can't resolve the `tsx` transpiler Payload uses internally
- Error: `Cannot find module 'tsx://...'`

### Workaround: --disable-transpile flag

```bash
# Force Bun runtime with --disable-transpile
bunx --bun payload migrate --disable-transpile
bunx --bun payload run src/seed.ts --disable-transpile
```

This disables `tsx` and lets Bun handle TypeScript transpilation itself.

### Recommended approach

**Use Bun as package manager, Node.js as runtime for Payload:**

```bash
# Install dependencies (Bun - fast)
bun install

# Dev server (works - Next.js uses Node internally)
bun run dev

# Payload CLI commands (safe - uses Node via bun run, not bun runtime)
bun run payload migrate:create
bun run payload generate:types

# If you MUST use Bun runtime for CLI
bunx --bun payload migrate --disable-transpile
```

**Important:** This applies to the Payload backend only. The Astro frontend has full Bun support.

**User note:** Use Bun as the default package manager for everything (Astro, daily work, installing packages). Keep pnpm available in the Payload backend for CLI commands where Node.js runtime is required (migrations, type generation). The backend `package.json` keeps its pnpm engine config for this reason. From the root, `bun run dev:backend` works fine since it delegates to Next.js which uses Node internally.

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
