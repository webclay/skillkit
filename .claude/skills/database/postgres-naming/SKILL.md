---
name: postgres-naming
description: Use this skill when creating or reviewing PostgreSQL database schemas, tables, columns, indexes, or constraints. Enforces snake_case naming in the database layer and camelCase mapping in the TypeScript/JavaScript codebase. Trigger words - database naming, table names, column names, schema conventions, postgres naming, database conventions, naming standards.
---

# PostgreSQL Naming Conventions

Standard naming conventions for PostgreSQL databases used in mobile and web apps. These rules apply to ALL database skills (Prisma, Drizzle, Supabase, Neon, etc.) when the database is PostgreSQL.

## The Core Rule

**Database layer:** `snake_case` everywhere
**Codebase layer:** `camelCase` everywhere

ORMs handle the mapping automatically. Define snake_case column names in the database, use camelCase property names in TypeScript.

## When to Use This Skill

- Creating new database tables or schemas
- Reviewing existing schema definitions
- Setting up migrations
- Adding columns, indexes, or constraints
- Any Prisma, Drizzle, or raw SQL schema work targeting PostgreSQL

## Why snake_case in PostgreSQL

PostgreSQL converts all unquoted identifiers to lowercase. Using snake_case avoids the need for quoted identifiers and matches PostgreSQL's internal conventions. Mixed case forces you to quote every identifier, which creates bugs and friction.

## Naming Rules

### Tables

- **Plural nouns** representing collections: `users`, `products`, `order_items`
- All lowercase with underscores
- No prefixes like `tbl_` or `t_`

```sql
-- Correct
CREATE TABLE users ( ... );
CREATE TABLE order_items ( ... );
CREATE TABLE product_categories ( ... );

-- Wrong
CREATE TABLE User ( ... );
CREATE TABLE tbl_users ( ... );
CREATE TABLE orderItems ( ... );
```

### Columns

- **Singular names** representing individual attributes
- All lowercase with underscores
- No prefixes like `col_` or `f_`
- Be descriptive: `year_founded` not `year`, `display_name` not `name` (when ambiguous)

```sql
-- Correct
email TEXT NOT NULL,
created_at TIMESTAMPTZ DEFAULT now(),
is_active BOOLEAN DEFAULT true,
year_founded INTEGER

-- Wrong
Email TEXT,
createdAt TIMESTAMPTZ,
isActive BOOLEAN,
col_year INTEGER
```

### Primary Keys

- Use `id` for simple auto-increment or UUID primary keys
- Use `uuid` type for distributed systems or public-facing IDs
- Use `serial`/`bigserial` for internal-only sequential IDs

```sql
id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
-- or
id BIGSERIAL PRIMARY KEY
```

### Foreign Keys

- Pattern: `{referenced_table_singular}_id`
- Always references the singular form of the parent table

```sql
-- Correct
user_id INTEGER REFERENCES users(id),
product_category_id INTEGER REFERENCES product_categories(id),
parent_comment_id INTEGER REFERENCES comments(id)

-- Wrong
userId INTEGER,
users_id INTEGER,
fk_user INTEGER
```

### Indexes

- Pattern: `idx_{table}_{column(s)}`
- For composite indexes, list columns in order

```sql
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_user_id_created_at ON orders(user_id, created_at);
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);
```

### Constraints

- Unique: `uq_{table}_{column(s)}`
- Check: `ck_{table}_{description}`
- Foreign key: `fk_{table}_{referenced_table}`

```sql
ALTER TABLE users ADD CONSTRAINT uq_users_email UNIQUE (email);
ALTER TABLE orders ADD CONSTRAINT ck_orders_amount_positive CHECK (amount > 0);
ALTER TABLE orders ADD CONSTRAINT fk_orders_users FOREIGN KEY (user_id) REFERENCES users(id);
```

### Boolean Columns

- Prefix with `is_`, `has_`, or `can_`

```sql
is_active BOOLEAN DEFAULT true,
is_verified BOOLEAN DEFAULT false,
has_subscription BOOLEAN DEFAULT false,
can_publish BOOLEAN DEFAULT false
```

### Timestamps

- Use `_at` suffix: `created_at`, `updated_at`, `deleted_at`, `verified_at`
- Always use `TIMESTAMPTZ` (with timezone), never `TIMESTAMP`

```sql
created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
deleted_at TIMESTAMPTZ
```

### Junction Tables (Many-to-Many)

- Pattern: `{table_a}_{table_b}` in alphabetical order
- Contains foreign keys to both tables

```sql
CREATE TABLE categories_products (
  category_id INTEGER REFERENCES categories(id),
  product_id INTEGER REFERENCES products(id),
  PRIMARY KEY (category_id, product_id)
);
```

### Enums

- Type name: singular `snake_case`
- Values: `snake_case`

```sql
CREATE TYPE order_status AS ENUM ('pending', 'processing', 'shipped', 'delivered', 'cancelled');
CREATE TYPE user_role AS ENUM ('admin', 'editor', 'viewer');
```

## Data Types

Use PostgreSQL's native types:

| Use | Instead of |
|-----|------------|
| `TEXT` | `VARCHAR(n)` (unless you need a hard limit) |
| `UUID` | `TEXT` for IDs |
| `TIMESTAMPTZ` | `TIMESTAMP` |
| `BOOLEAN` | `INTEGER` (0/1) |
| `JSONB` | `JSON` (JSONB is indexed, JSON is not) |
| `BIGINT` | `INTEGER` (for IDs that may grow large) |

## ORM Mapping Examples

### Drizzle

Drizzle lets you define snake_case DB names with camelCase TS properties:

```ts
export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  displayName: text('display_name').notNull(),
  isActive: boolean('is_active').default(true),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
  updatedAt: timestamp('updated_at', { withTimezone: true }).defaultNow().notNull(),
});

export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: text('title').notNull(),
  authorId: integer('author_id').references(() => users.id),
  publishedAt: timestamp('published_at', { withTimezone: true }),
  isPublished: boolean('is_published').default(false),
  createdAt: timestamp('created_at', { withTimezone: true }).defaultNow().notNull(),
});
```

### Prisma

Prisma uses `@map` and `@@map` to separate DB names from model names:

```prisma
model User {
  id          Int      @id @default(autoincrement())
  displayName String   @map("display_name")
  isActive    Boolean  @default(true) @map("is_active")
  createdAt   DateTime @default(now()) @map("created_at") @db.Timestamptz
  updatedAt   DateTime @updatedAt @map("updated_at") @db.Timestamptz
  posts       Post[]

  @@map("users")
  @@index([email])
}

model Post {
  id          Int       @id @default(autoincrement())
  title       String
  authorId    Int       @map("author_id")
  author      User      @relation(fields: [authorId], references: [id])
  isPublished Boolean   @default(false) @map("is_published")
  publishedAt DateTime? @map("published_at") @db.Timestamptz
  createdAt   DateTime  @default(now()) @map("created_at") @db.Timestamptz

  @@map("posts")
  @@index([authorId])
}
```

## Anti-Patterns

| Anti-Pattern | Why It's Wrong | Correct |
|--------------|----------------|---------|
| `tbl_users`, `col_email` | Hungarian notation adds noise | `users`, `email` |
| `"UserProfile"` (quoted) | Requires quotes everywhere, breaks tooling | `user_profiles` |
| `orderItems` (camelCase in DB) | PostgreSQL lowercases unquoted identifiers | `order_items` |
| `_AmenityToBar` | Auto-generated names obscure purpose | `amenities_bars` |
| `data`, `value`, `type` | Too generic, no context | `order_total`, `preference_value`, `account_type` |
| `TIMESTAMP` without timezone | Loses timezone info, causes bugs | `TIMESTAMPTZ` |

## Checklist for New Tables

Before creating any new table, verify:

- [ ] Table name is plural and snake_case
- [ ] All columns are snake_case
- [ ] Foreign keys follow `{table_singular}_id` pattern
- [ ] Boolean columns have `is_`, `has_`, or `can_` prefix
- [ ] Timestamps use `_at` suffix and `TIMESTAMPTZ` type
- [ ] `created_at` and `updated_at` are included
- [ ] Indexes follow `idx_{table}_{columns}` pattern
- [ ] ORM properties are camelCase, mapped to snake_case DB names
- [ ] No Hungarian notation prefixes
- [ ] No quoted identifiers needed
