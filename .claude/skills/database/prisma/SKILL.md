---
name: prisma
description: Use this skill when working with Prisma ORM for database schema design, migrations, queries, or relations. Activate when the user mentions Prisma, database models, schema changes, migration issues, or Prisma Client operations like findMany, create, or update.
---

# Prisma

Type-safe database access with auto-generated types, intuitive queries, migrations, and visual database browser. Supports PostgreSQL, MySQL, SQLite, MongoDB, and SQL Server.

## When to Use This Skill

- User asks to add database functionality
- package.json contains "prisma" or "@prisma/client"
- User mentions database, schema, migrations, or ORM
- Working with .prisma files
- User asks about queries, relations, or data modeling

## Instructions

### Step 1: Install Prisma

```bash
pnpm add prisma @prisma/client -D
npx prisma init
```

### Step 2: Configure Database

**.env:**
```env
DATABASE_URL="postgresql://user:password@localhost:5432/mydb"
```

**prisma/schema.prisma:**
```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client"  // v7 syntax
}
```

### Step 3: Define Schema

```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([email])
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  authorId  Int
  author    User     @relation(fields: [authorId], references: [id])
  createdAt DateTime @default(now())

  @@index([authorId])
}
```

### Step 4: Run Migrations

```bash
npx prisma migrate dev --name init
npx prisma generate
```

### Step 5: Create Prisma Client

**lib/db.ts:**
```ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const db = globalForPrisma.prisma ?? new PrismaClient();

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = db;
```

## Framework Integration

### TanStack Start - API Routes (REQUIRED)

**CRITICAL:** In TanStack Start, ALL Prisma queries MUST go in API routes (`app/routes/api/*`), NOT in loaders or components. This prevents Prisma from leaking into the client bundle.

**Correct - API Route:**
```ts
// app/routes/api/users.ts
import { createFileRoute } from '@tanstack/react-router';
import { db } from '@/lib/db';

export const Route = createFileRoute('/api/users')({
  server: {
    handlers: {
      GET: async () => {
        const users = await db.user.findMany({
          include: { posts: true },
        });
        return Response.json({ users });
      },
      POST: async ({ request }) => {
        const body = await request.json();
        const user = await db.user.create({ data: body });
        return Response.json({ user });
      },
    },
  },
});
```

**Frontend - Fetch from API route:**
```tsx
// app/routes/users.tsx
import { createFileRoute } from '@tanstack/react-router';

export const Route = createFileRoute('/users')({
  loader: async () => {
    const response = await fetch('/api/users');
    const { users } = await response.json();
    return { users };
  },
  component: Users,
});

function Users() {
  const { users } = Route.useLoaderData();
  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

**Wrong - DON'T import db in loaders:**
```ts
// ❌ WRONG - This leaks Prisma to client bundle
import { db } from '@/lib/db';

export const Route = createFileRoute('/users')({
  loader: async () => {
    const users = await db.user.findMany();  // DON'T DO THIS
    return { users };
  },
});
```

### Next.js - Server Actions/API Routes

**Server Action:**
```ts
'use server';
import { db } from '@/lib/db';

export async function getUsers() {
  return await db.user.findMany();
}
```

**API Route:**
```ts
// app/api/users/route.ts
import { db } from '@/lib/db';
import { NextResponse } from 'next/server';

export async function GET() {
  const users = await db.user.findMany();
  return NextResponse.json({ users });
}
```

## Query Examples

**Create:**
```ts
const user = await db.user.create({
  data: { email: 'alice@example.com', name: 'Alice' },
});
```

**Read:**
```ts
const users = await db.user.findMany({
  where: { name: { contains: 'alice' } },
  include: { posts: true },
});
```

**Update:**
```ts
const user = await db.user.update({
  where: { id: 1 },
  data: { name: 'Alice Updated' },
});
```

**Delete:**
```ts
await db.user.delete({ where: { id: 1 } });
```

**With Relations:**
```ts
const userWithPosts = await db.user.findUnique({
  where: { id: 1 },
  include: {
    posts: {
      where: { published: true },
      orderBy: { createdAt: 'desc' },
    },
  },
});
```

**Pagination:**
```ts
const users = await db.user.findMany({
  skip: 10,
  take: 10,
  orderBy: { createdAt: 'desc' },
});
```

## Reference Files

- [schema.md](schema.md) - Schema definition patterns
- [queries.md](queries.md) - Advanced query examples
- [migrations.md](migrations.md) - Migration workflow
- [relations.md](relations.md) - Relation patterns

## Common Commands

```bash
npx prisma migrate dev --name [name]  # Create migration
npx prisma migrate deploy             # Apply migrations (production)
npx prisma generate                   # Generate client
npx prisma studio                     # Visual database browser
npx prisma db push                    # Push schema without migration
```

## Prisma 7 with Neon Adapter

In Prisma 7, `PrismaNeon` takes a **config object**, NOT a Pool instance.

**lib/db.ts:**
```ts
import { PrismaNeon } from "@prisma/adapter-neon";
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient };

function createPrismaClient() {
  const connectionString = process.env.DATABASE_URL;
  if (!connectionString) {
    throw new Error("DATABASE_URL environment variable is not set");
  }
  // PrismaNeon takes PoolConfig, not a Pool instance
  const adapter = new PrismaNeon({ connectionString });
  return new PrismaClient({
    adapter,
    log: process.env.NODE_ENV === "development" ? ["error", "warn"] : ["error"],
  });
}

export const db = globalForPrisma.prisma || createPrismaClient();

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = db;
```

**Wrong (old Prisma 5/6 pattern):**
```ts
// DON'T DO THIS
import { Pool } from "@neondatabase/serverless";
const pool = new Pool({ connectionString: process.env.DATABASE_URL });
const adapter = new PrismaNeon(pool);  // Wrong - don't pass Pool
```

## Prisma Config for CLI Commands

`prisma.config.ts` needs `dotenv/config` import and explicit `datasource.url` for CLI commands to work:

```ts
import "dotenv/config";
import path from "node:path";
import { defineConfig } from "prisma/config";

export default defineConfig({
  earlyAccess: true,
  schema: path.join(__dirname, "prisma/schema.prisma"),
  datasource: {
    url: process.env.DATABASE_URL!,
  },
  migrate: {
    adapter: async () => {
      const { PrismaNeon } = await import("@prisma/adapter-neon");
      return new PrismaNeon({ connectionString: process.env.DATABASE_URL! });
    },
  },
});
```

**Note:** `@neondatabase/serverless` is no longer needed as a direct dependency when using `@prisma/adapter-neon` v7+.

## Tips

- Use singleton pattern for client (prevents connection exhaustion)
- Add `@@index` for frequently queried fields
- Use `select` to fetch only needed fields
- Use cursor pagination for large datasets
- Check https://www.prisma.io/docs for latest v7 features

## How to Verify

### Quick Checks
- `npx prisma validate` - Schema syntax is correct
- `npx prisma db push --dry-run` - Schema can be applied to database
- `npx prisma generate` - Client generates without errors

### Manual Verification
- Run `npx prisma studio` and confirm tables exist
- Create a test record through Studio and verify it saves
- Query the record in your app to confirm db connection works

### Common Issues
- "Can't reach database": Check DATABASE_URL and network access
- "Relation not found": Run `npx prisma generate` after schema changes
- Type errors: Regenerate client with `npx prisma generate`
- "datasource.url property is required": Add `import "dotenv/config"` and `datasource: { url }` to `prisma.config.ts`
- "PrismaNeon is not a constructor" or type errors: Pass config object `{ connectionString }` directly to PrismaNeon, not a Pool instance (Prisma 7 change)
