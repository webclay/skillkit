# Access Control, Hooks, Types, and Versions

Reference for Payload CMS access control, lifecycle hooks, auto-generated types, and versioning/drafts.

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
console.log(car.title); // ok
console.log(car.manufacturer.name); // ok
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
