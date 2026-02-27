# REST API, Local API, and Common Patterns

Reference for Payload's auto-generated REST API, Local API for Next.js integration, Select API, and common collection patterns.

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
