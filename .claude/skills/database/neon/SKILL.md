---
name: neon
description: Neon serverless Postgres with branching and autoscaling. Trigger words - neon, serverless postgres, database branching, branch database, scale to zero, neon db, preview database
---

# Neon

Serverless PostgreSQL with instant provisioning, database branching, and autoscaling.

## When to Use This Skill

- Building serverless applications
- Need database per branch/PR (preview environments)
- Want Git-like branching for databases
- Require automatic scaling and scale-to-zero
- Cost optimization for development databases

## When NOT to Use

- Need always-on dedicated database
- Require self-hosted Postgres
- Heavy write workloads needing predictable latency

## Setup

```bash
npm i @neondatabase/serverless
```

### Environment Variables

```env
# From Neon Console
DATABASE_URL=postgresql://user:pass@ep-cool-123-pooler.us-east-2.aws.neon.tech/neondb?sslmode=require

# For migrations (direct connection)
DATABASE_URL_UNPOOLED=postgresql://user:pass@ep-cool-123.us-east-2.aws.neon.tech/neondb?sslmode=require
```

## Serverless Driver (Edge Functions)

```typescript
import { neon } from '@neondatabase/serverless';

const sql = neon(process.env.DATABASE_URL!);

// Simple queries
const users = await sql`SELECT * FROM users WHERE active = true`;

// Parameterized (prevents SQL injection)
const user = await sql`
  SELECT * FROM users WHERE id = ${userId}
`;

// Insert with returning
const [newUser] = await sql`
  INSERT INTO users (name, email)
  VALUES (${name}, ${email})
  RETURNING *
`;

// Transactions
await sql.transaction([
  sql`INSERT INTO users (name) VALUES ('Alice')`,
  sql`INSERT INTO posts (user_id, title) VALUES (1, 'First Post')`,
]);
```

## Next.js API Route

```typescript
// app/api/users/route.ts
import { neon } from '@neondatabase/serverless';
import { NextRequest, NextResponse } from 'next/server';

const sql = neon(process.env.DATABASE_URL!);

export async function GET() {
  const users = await sql`
    SELECT id, name, email FROM users
    ORDER BY created_at DESC
    LIMIT 100
  `;
  return NextResponse.json({ users });
}

export async function POST(request: NextRequest) {
  const { name, email } = await request.json();

  const [user] = await sql`
    INSERT INTO users (name, email)
    VALUES (${name}, ${email})
    RETURNING *
  `;

  return NextResponse.json({ user }, { status: 201 });
}
```

## Server Component

```tsx
// app/users/page.tsx
import { neon } from '@neondatabase/serverless';

const sql = neon(process.env.DATABASE_URL!);

export default async function UsersPage() {
  const users = await sql`SELECT * FROM users ORDER BY created_at DESC`;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

## Prisma Integration

```prisma
// prisma/schema.prisma
datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")          // Pooled for queries
  directUrl = env("DATABASE_URL_UNPOOLED") // Direct for migrations
}
```

### Prisma 7 with Neon Adapter

In Prisma 7, `PrismaNeon` takes a **config object**, NOT a Pool instance.

**Correct (Prisma 7+):**
```typescript
import { PrismaNeon } from "@prisma/adapter-neon";
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient };

function createPrismaClient() {
  const connectionString = process.env.DATABASE_URL;
  if (!connectionString) {
    throw new Error("DATABASE_URL environment variable is not set");
  }
  // PrismaNeon takes PoolConfig directly, not a Pool instance
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
```typescript
// DON'T DO THIS - this was the old pattern
import { Pool } from "@neondatabase/serverless";
const pool = new Pool({ connectionString: process.env.DATABASE_URL });
const adapter = new PrismaNeon(pool);  // Wrong - don't pass Pool
```

**Note:** `@neondatabase/serverless` is no longer needed as a direct dependency when using `@prisma/adapter-neon` v7+.

### Prisma Config for CLI Commands

`prisma.config.ts` needs `dotenv/config` import for CLI commands to work:

```typescript
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

Run migrations with direct connection:

```bash
npx prisma migrate dev
```

## Database Branching

```bash
# Create branch from main
neonctl branches create --name dev --parent main

# Create feature branch
neonctl branches create --name feature/auth --parent main

# Get connection string for branch
neonctl connection-string --branch dev

# Delete branch
neonctl branches delete feature/auth
```

### Preview Environments (Vercel)

```yaml
# .github/workflows/preview.yml
- name: Create Neon branch
  run: |
    BRANCH_NAME="preview-${{ github.event.pull_request.number }}"
    neonctl branches create --name $BRANCH_NAME
    CONNECTION_STRING=$(neonctl connection-string --branch $BRANCH_NAME)
    echo "DATABASE_URL=$CONNECTION_STRING" >> $GITHUB_ENV
```

## pgvector (AI Embeddings)

```sql
-- Enable extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Create table with embeddings
CREATE TABLE documents (
  id SERIAL PRIMARY KEY,
  content TEXT,
  embedding vector(1536)  -- OpenAI embedding size
);

-- Create index for fast search
CREATE INDEX ON documents USING hnsw (embedding vector_cosine_ops);
```

```typescript
// Find similar documents
async function findSimilar(queryEmbedding: number[]) {
  return await sql`
    SELECT content,
           1 - (embedding <=> ${JSON.stringify(queryEmbedding)}::vector) AS similarity
    FROM documents
    ORDER BY embedding <=> ${JSON.stringify(queryEmbedding)}::vector
    LIMIT 5
  `;
}
```

## Connection Pooling

Use `-pooler` suffix for serverless:

```bash
# Pooled (for serverless functions)
DATABASE_URL=postgresql://user:pass@ep-123-pooler.region.neon.tech/db

# Direct (for migrations, long connections)
DATABASE_URL_UNPOOLED=postgresql://user:pass@ep-123.region.neon.tech/db
```

## Autoscaling Config

In Neon Console → Project Settings → Compute:

| Environment | Min | Max | Scale-to-zero |
|-------------|-----|-----|---------------|
| Development | 0 | 0.5 | Yes |
| Staging | 0.25 | 1 | Yes |
| Production | 0.5 | 4 | No |

## Tips

- Always use pooled connection for serverless functions
- Use direct connection only for migrations
- Create branches for each PR/feature
- Enable SSL: `?sslmode=require`

## How to Verify

### Quick Checks
- Query returns data in server component
- Branches visible in Neon Console
- Migrations run without connection errors

### Common Issues
- "Connection refused": Check connection string, use pooler for serverless
- Migration timeout: Use `DATABASE_URL_UNPOOLED` for migrations
- Cold start latency: Scale-to-zero adds ~500ms first query
- "PrismaNeon is not a constructor": Using old Prisma 5/6 pattern - pass `{ connectionString }` config object, not a Pool instance
- "datasource.url property is required": Add `import "dotenv/config"` to `prisma.config.ts` for CLI commands
