# Better Auth Database Schema

## Critical: ID Requirements

Better Auth generates its own **string IDs** (not UUIDs). Do **NOT** use `@db.Uuid` or `@default(uuid())` on Better Auth tables. Using these will cause "invalid input syntax for type uuid" errors.

## Prisma Schema

Add to your `prisma/schema.prisma`:

```prisma
model user {
  id            String    @id  // No @default() - Better Auth generates IDs
  email         String    @unique
  name          String?
  emailVerified Boolean?  @map("email_verified")
  image         String?
  createdAt     DateTime  @default(now()) @map("created_at")
  updatedAt     DateTime  @updatedAt @map("updated_at")

  sessions session[]
  accounts account[]
}

model session {
  id        String   @id  // No @default() - Better Auth generates IDs
  userId    String   @map("user_id")
  token     String   @unique
  expiresAt DateTime @map("expires_at")
  ipAddress String?  @map("ip_address")
  userAgent String?  @map("user_agent")
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  user user @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
}

model account {
  id                    String    @id  // No @default() - Better Auth generates IDs
  userId                String    @map("user_id")
  accountId             String    @map("account_id")
  providerId            String    @map("provider_id")
  accessToken           String?   @map("access_token")
  refreshToken          String?   @map("refresh_token")
  accessTokenExpiresAt  DateTime? @map("access_token_expires_at")
  refreshTokenExpiresAt DateTime? @map("refresh_token_expires_at")
  scope                 String?
  idToken               String?   @map("id_token")
  password              String?
  createdAt             DateTime  @default(now()) @map("created_at")
  updatedAt             DateTime  @updatedAt @map("updated_at")

  user user @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([providerId, accountId])
  @@index([userId])
}

model verification {
  id         String   @id  // No @default() - Better Auth generates IDs
  identifier String
  value      String
  expiresAt  DateTime @map("expires_at")
  createdAt  DateTime @default(now()) @map("created_at")
  updatedAt  DateTime @updatedAt @map("updated_at")

  @@index([identifier])
}
```

## Your Application Tables

For your own tables (not Better Auth tables), you CAN use `@default(uuid())` but still omit `@db.Uuid`:

```prisma
model quizzes {
  id      String @id @default(uuid())  // OK for your own tables
  userId  String @map("user_id")
  title   String
  // ...
}
```

## Prisma Client Singleton

For serverless (prevents connection pool exhaustion):

```ts
// lib/db.ts
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const db = globalForPrisma.prisma ?? new PrismaClient({
  log: process.env.NODE_ENV === "development" ? ["error", "warn"] : ["error"],
});

if (process.env.NODE_ENV !== "production") {
  globalForPrisma.prisma = db;
}
```

## Neon Adapter (Serverless PostgreSQL)

```ts
import { PrismaNeon } from "@prisma/adapter-neon";
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

function createPrismaClient() {
  const adapter = new PrismaNeon({
    connectionString: process.env.DATABASE_URL,
  });

  return new PrismaClient({
    adapter,
    log: process.env.NODE_ENV === "development" ? ["error", "warn"] : ["error"],
  });
}

export const db = globalForPrisma.prisma ?? createPrismaClient();

if (process.env.NODE_ENV !== "production") {
  globalForPrisma.prisma = db;
}
```

## Run Migrations

```bash
npx prisma db push
# or
npx prisma migrate dev
```

## Direct PostgreSQL (Without ORM)

Better Auth can connect directly to PostgreSQL using the `pg` package, without requiring Prisma or Drizzle.

### Setup

```ts
// auth.ts
import { betterAuth } from "better-auth";
import { Pool } from "pg";

export const auth = betterAuth({
  database: new Pool({
    connectionString: "postgres://user:password@localhost:5432/database",
  }),
});
```

### Schema Generation and Migration

Better Auth CLI can generate and migrate schemas directly:

```bash
# Generate schema
pnpm dlx @better-auth/cli@latest generate

# Run migrations
pnpm dlx @better-auth/cli@latest migrate
```

### Using a Non-Default Schema

To use a schema other than `public` (e.g., `auth`):

**Option 1: Connection string (Recommended)**
```ts
export const auth = betterAuth({
  database: new Pool({
    connectionString: "postgres://user:password@localhost:5432/database?options=-c search_path=auth",
  }),
});
```

**Option 2: Pool options**
```ts
export const auth = betterAuth({
  database: new Pool({
    host: "localhost",
    port: 5432,
    user: "postgres",
    password: "password",
    database: "my-db",
    options: "-c search_path=auth",
  }),
});
```

**Prerequisites for non-default schema:**
```sql
-- Create the schema
CREATE SCHEMA IF NOT EXISTS auth;

-- Grant permissions
GRANT ALL PRIVILEGES ON SCHEMA auth TO your_user;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA auth TO your_user;
ALTER DEFAULT PRIVILEGES IN SCHEMA auth GRANT ALL ON TABLES TO your_user;
```

### Experimental: Database Joins

Enable joins for 2-3x performance improvements on endpoints like `/get-session`:

```ts
export const auth = betterAuth({
  experimental: { joins: true }
});
```

Note: You may need to run migrations after enabling this feature.

### Reference

- [PostgreSQL Adapter Docs](https://www.better-auth.com/docs/adapters/postgresql)
- [Kysely PostgresDialect](https://kysely-org.github.io/kysely-apidoc/classes/PostgresDialect.html)
