# Using an ORM with Supabase

You have three options for querying your Supabase database:

## Option 1: Supabase JS Client (Recommended for most cases)

Use the built-in `@supabase/supabase-js` client. This is the simplest option and integrates well with Supabase Auth and RLS.

## Option 2: Prisma ORM

If you prefer Prisma's schema-based approach and migration tooling, you need to install an adapter.

**Install packages:**
```bash
pnpm add @prisma/adapter-pg pg
pnpm add -D prisma @types/pg
```

**Get connection strings from Supabase:**
1. Go to Supabase Dashboard -> Connect -> ORMs -> Prisma
2. Copy both connection strings (pooled and direct)

**Environment variables:**
```env
# Pooled connection (for queries) - has ?pgbouncer=true
DATABASE_URL="postgresql://postgres.[project-ref]:[password]@aws-0-[region].pooler.supabase.com:6543/postgres?pgbouncer=true"

# Direct connection (for migrations)
DIRECT_URL="postgresql://postgres.[project-ref]:[password]@aws-0-[region].pooler.supabase.com:5432/postgres"
```

**prisma/schema.prisma:**
```prisma
datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
}

generator client {
  provider = "prisma-client"
}
```

**lib/db.ts:**
```ts
import { PrismaClient } from "@prisma/client";
import { PrismaPg } from "@prisma/adapter-pg";
import { Pool } from "pg";

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient };

function createPrismaClient() {
  const pool = new Pool({ connectionString: process.env.DATABASE_URL });
  const adapter = new PrismaPg(pool);
  return new PrismaClient({ adapter });
}

export const db = globalForPrisma.prisma || createPrismaClient();

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = db;
```

## Option 3: Drizzle ORM

If you prefer Drizzle's SQL-like syntax and lighter footprint.

**Install packages:**
```bash
pnpm add drizzle-orm postgres
pnpm add -D drizzle-kit
```

**Get connection string from Supabase:**
1. Go to Supabase Dashboard -> Connect -> ORMs
2. Copy the connection string

**Environment variables:**
```env
DATABASE_URL="postgresql://postgres.[project-ref]:[password]@aws-0-[region].pooler.supabase.com:6543/postgres"
```

**lib/db.ts:**
```ts
import { drizzle } from "drizzle-orm/postgres-js";
import postgres from "postgres";
import * as schema from "./schema";

const client = postgres(process.env.DATABASE_URL!);
export const db = drizzle(client, { schema });
```

**drizzle.config.ts:**
```ts
import type { Config } from "drizzle-kit";

export default {
  schema: "./lib/schema.ts",
  out: "./drizzle",
  dialect: "postgresql",
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
} satisfies Config;
```

## Which option should I choose?

| Option | Best for |
|--------|----------|
| **Supabase JS** | Most projects - simple queries, using Supabase Auth/RLS |
| **Prisma** | Teams wanting visual Studio browser, robust migrations, higher-level abstraction |
| **Drizzle** | Developers comfortable with SQL, edge/serverless deployments, smaller bundle size |
