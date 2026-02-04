---
name: drizzle
description: Drizzle ORM database patterns and implementation. Trigger words - drizzle, database, schema, migration, table, query, select, insert, update, delete, edge database, serverless db
---

# Drizzle ORM

Lightweight, TypeScript-first ORM with SQL-like syntax. Zero dependencies, edge-ready, significantly faster than Prisma.

## When to Use This Skill

- User asks about Drizzle ORM setup
- package.json contains "drizzle-orm" or "drizzle-kit"
- User mentions SQL-first ORM or edge database
- Working with Supabase, Neon, Turso, PlanetScale, or SQLite
- User wants lightweight alternative to Prisma

## Instructions

### Step 1: Install Drizzle

Choose the driver based on your database provider:

**PostgreSQL (Supabase):**
```bash
pnpm add drizzle-orm postgres
pnpm add -D drizzle-kit
```

**PostgreSQL (Neon):**
```bash
pnpm add drizzle-orm @neondatabase/serverless
pnpm add -D drizzle-kit
```

**SQLite (Turso):**
```bash
pnpm add drizzle-orm @libsql/client
pnpm add -D drizzle-kit
```

### Step 2: Create Database Connection

**lib/db.ts (Supabase):**
```ts
import { drizzle } from "drizzle-orm/postgres-js";
import postgres from "postgres";
import * as schema from "./schema";

const client = postgres(process.env.DATABASE_URL!);
export const db = drizzle(client, { schema });
```

**lib/db.ts (Neon):**
```ts
import { drizzle } from 'drizzle-orm/neon-http';
import { neon } from '@neondatabase/serverless';
import * as schema from './schema';

const sql = neon(process.env.DATABASE_URL!);
export const db = drizzle(sql, { schema });
```

### Step 3: Define Schema

**lib/schema.ts:**
```ts
import { pgTable, serial, text, timestamp, boolean, integer } from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: text('email').notNull().unique(),
  name: text('name').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  content: text('content'),
  authorId: integer('author_id').references(() => users.id),
  createdAt: timestamp('created_at').defaultNow(),
});

export const usersRelations = relations(users, ({ many }) => ({
  posts: many(posts),
}));

export const postsRelations = relations(posts, ({ one }) => ({
  author: one(users, {
    fields: [posts.authorId],
    references: [users.id],
  }),
}));
```

### Step 4: Configure Drizzle Kit

**drizzle.config.ts:**
```ts
import type { Config } from 'drizzle-kit';

export default {
  schema: './lib/schema.ts',
  out: './drizzle',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
} satisfies Config;
```

### Step 5: Run Migrations

```bash
npx drizzle-kit generate
npx drizzle-kit push
```

## Examples

**Select:**
```ts
import { eq } from 'drizzle-orm';

const allUsers = await db.select().from(users);

const user = await db.select()
  .from(users)
  .where(eq(users.email, 'alice@example.com'));
```

**Insert:**
```ts
const newUser = await db.insert(users)
  .values({ email: 'alice@example.com', name: 'Alice' })
  .returning();
```

**Update:**
```ts
const updated = await db.update(users)
  .set({ name: 'Alice Updated' })
  .where(eq(users.id, 1))
  .returning();
```

**Delete:**
```ts
await db.delete(users).where(eq(users.id, 1));
```

**With Relations:**
```ts
const userWithPosts = await db.query.users.findFirst({
  where: eq(users.id, 1),
  with: { posts: true },
});
```

**Transactions:**
```ts
await db.transaction(async (tx) => {
  const user = await tx.insert(users).values({ ... }).returning();
  await tx.insert(posts).values({ authorId: user[0].id, ... });
});
```

## Reference Files

- [schema.md](schema.md) - Schema definition patterns
- [queries.md](queries.md) - Advanced query examples
- [migrations.md](migrations.md) - Migration workflow

## Common Commands

```bash
npx drizzle-kit generate    # Create migration
npx drizzle-kit push        # Apply to database
npx drizzle-kit studio      # Visual database browser
```

## Database Provider Setup

### Supabase

1. Go to Supabase Dashboard → Connect → ORMs
2. Copy the connection string

```env
DATABASE_URL="postgresql://postgres.[project-ref]:[password]@aws-0-[region].pooler.supabase.com:6543/postgres"
```

### Neon

1. Go to Neon Console → Connection Details
2. Copy the pooled connection string

```env
DATABASE_URL="postgresql://user:pass@ep-cool-123-pooler.us-east-2.aws.neon.tech/neondb?sslmode=require"
```

## Tips

- Use `drizzle-kit studio` for visual database exploration
- Define relations for relational queries with `with:`
- Use `InferSelectModel<typeof users>` for TypeScript types
- Drizzle is SQL-first - familiar to SQL developers
- Different databases need different drivers - check Step 1 for the correct packages

## How to Verify

### Quick Checks
- `npx drizzle-kit check` - Schema is valid and consistent
- `npx drizzle-kit push --dry-run` - See what changes would be applied
- TypeScript compiles without errors on schema imports

### Manual Verification
- Run `npx drizzle-kit studio` and confirm tables exist
- Insert a test record and verify it appears in Studio
- Run a simple query in your app to confirm connection works

### Common Issues
- "Connection refused": Check DATABASE_URL format and network access
- "Table doesn't exist": Run `npx drizzle-kit push` to sync schema
- Import errors: Ensure schema file exports all tables
