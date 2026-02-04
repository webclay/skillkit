# Prisma Migrations Reference

## Commands

| Command | Use Case |
|---------|----------|
| `npx prisma migrate dev --name [name]` | Create and apply migration (development) |
| `npx prisma migrate deploy` | Apply pending migrations (production) |
| `npx prisma migrate reset` | Reset database and apply all migrations |
| `npx prisma migrate status` | Show migration status |
| `npx prisma db push` | Push schema without creating migration |
| `npx prisma generate` | Regenerate Prisma Client |

## Development Workflow

### 1. Modify Schema

```prisma
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  name  String?
  age   Int?    // NEW FIELD
}
```

### 2. Create Migration

```bash
npx prisma migrate dev --name add_user_age
```

This command:
1. Creates SQL migration file in `prisma/migrations/`
2. Applies migration to database
3. Regenerates Prisma Client

### 3. Verify

```bash
npx prisma studio  # Visual check
```

## Production Workflow

### Apply Migrations

```bash
# In CI/CD or deployment script
npx prisma migrate deploy
```

**Important:** Never use `migrate dev` in production!

### Typical Deploy Script

```json
{
  "scripts": {
    "postinstall": "prisma generate",
    "deploy": "prisma migrate deploy && node dist/server.js"
  }
}
```

## Migration Files

**Location:** `prisma/migrations/[timestamp]_[name]/migration.sql`

**Example:**
```sql
-- CreateTable
CREATE TABLE "User" (
    "id" SERIAL NOT NULL,
    "email" TEXT NOT NULL,
    "name" TEXT,
    CONSTRAINT "User_pkey" PRIMARY KEY ("id")
);

-- CreateIndex
CREATE UNIQUE INDEX "User_email_key" ON "User"("email");
```

## Common Scenarios

### Add Required Field to Existing Table

**Problem:** Can't add required field if table has data.

**Solution 1: Default value**
```prisma
model User {
  role String @default("USER")  // Add with default
}
```

**Solution 2: Three-step migration**
```bash
# 1. Add as optional
# schema: role String?
npx prisma migrate dev --name add_role_optional

# 2. Backfill data (manual SQL or script)
UPDATE users SET role = 'USER' WHERE role IS NULL;

# 3. Make required
# schema: role String
npx prisma migrate dev --name make_role_required
```

### Rename Field

**Warning:** Prisma treats this as drop + create.

**Solution:**
```bash
# 1. Create empty migration
npx prisma migrate dev --name rename_name_to_fullname --create-only

# 2. Edit the generated SQL:
# Change:
#   ALTER TABLE "User" DROP COLUMN "name";
#   ALTER TABLE "User" ADD COLUMN "fullName" TEXT;
# To:
#   ALTER TABLE "User" RENAME COLUMN "name" TO "fullName";

# 3. Apply migration
npx prisma migrate dev
```

### Rename Table

```prisma
model User {
  // ...fields
  @@map("users")  // Keep old table name in database
}
```

### Add Index

```prisma
model Post {
  authorId Int
  // ...
  @@index([authorId])
}
```

### Add Unique Constraint

```prisma
model User {
  email String @unique
}

// Composite unique
model PostTag {
  postId Int
  tagId  Int
  @@unique([postId, tagId])
}
```

## Handling Migration Conflicts

### Reset Development Database

```bash
npx prisma migrate reset
```

This will:
1. Drop database
2. Create database
3. Apply all migrations
4. Seed if configured

### Resolve Drift

When database schema differs from migrations:

```bash
# Create baseline from current database
npx prisma migrate diff \
  --from-schema-datamodel prisma/schema.prisma \
  --to-schema-datasource prisma/schema.prisma \
  --script > fix.sql

# Apply fix
npx prisma db execute --file fix.sql
```

## Seeding

**prisma/seed.ts:**
```ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
  await prisma.user.upsert({
    where: { email: 'admin@example.com' },
    update: {},
    create: {
      email: 'admin@example.com',
      name: 'Admin',
      role: 'ADMIN',
    },
  });
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(() => prisma.$disconnect());
```

**package.json:**
```json
{
  "prisma": {
    "seed": "ts-node prisma/seed.ts"
  }
}
```

**Run seed:**
```bash
npx prisma db seed
```

## Connection Pooling for Migrations

For hosted databases with connection pooling:

```prisma
datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")           // Pooled URL
  directUrl = env("DATABASE_URL_UNPOOLED")  // Direct URL for migrations
}
```

**.env:**
```env
DATABASE_URL="postgresql://...?pgbouncer=true"
DATABASE_URL_UNPOOLED="postgresql://...direct..."
```

## Best Practices

1. **Always commit migrations** - Migration files are part of your codebase
2. **Never edit applied migrations** - Create new migrations for fixes
3. **Use meaningful names** - `add_user_profile`, not `migration1`
4. **Test migrations locally** - Before deploying to production
5. **Backup before production migrations** - Especially for destructive changes
6. **Use `--create-only` for complex changes** - Review SQL before applying
