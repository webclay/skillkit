# Collections and Field Types

Detailed reference for Payload CMS collections, all field types, media handling, and block-based layouts.

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

The Lexical editor is highly extensible (see rich-text-and-components.md).

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
