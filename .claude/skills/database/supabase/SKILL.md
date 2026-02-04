---
name: supabase
description: Supabase database, auth, and backend patterns. Trigger words - supabase, postgres, postgresql, RLS, row level security, edge function, realtime, supabase auth, supabase storage
---

# Supabase

Open-source Firebase alternative providing PostgreSQL database, authentication, storage, edge functions, and real-time subscriptions.

## When to Use This Skill

- User asks about Supabase setup or configuration
- package.json contains "@supabase/supabase-js" or "@supabase/ssr"
- User mentions PostgreSQL, RLS policies, or Row Level Security
- User asks about Supabase Auth or authentication
- Working with Edge Functions or real-time subscriptions

## Instructions

### Step 1: Install Supabase

```bash
pnpm add @supabase/supabase-js @supabase/ssr
```

### Step 2: Set Environment Variables

```env
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
```

### Step 3: Create Browser Client

**lib/supabase/client.ts:**
```ts
import { createBrowserClient } from '@supabase/ssr'

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}
```

### Step 4: Create Server Client

**lib/supabase/server.ts:**
```ts
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'

export async function createClient() {
  const cookieStore = await cookies()

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll()
        },
        setAll(cookiesToSet) {
          try {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, options)
            )
          } catch {
            // Can be ignored in Server Components
          }
        },
      },
    }
  )
}
```

### Step 5: Set Up Middleware

**middleware.ts:**
```ts
import { createServerClient } from '@supabase/ssr'
import { NextResponse, type NextRequest } from 'next/server'

export async function middleware(request: NextRequest) {
  let supabaseResponse = NextResponse.next({ request })

  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return request.cookies.getAll()
        },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value }) => request.cookies.set(name, value))
          supabaseResponse = NextResponse.next({ request })
          cookiesToSet.forEach(({ name, value, options }) =>
            supabaseResponse.cookies.set(name, value, options)
          )
        },
      },
    }
  )

  const { data: { user } } = await supabase.auth.getUser()

  if (!user && !request.nextUrl.pathname.startsWith('/login')) {
    const url = request.nextUrl.clone()
    url.pathname = '/login'
    return NextResponse.redirect(url)
  }

  return supabaseResponse
}

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)'],
}
```

## Examples

**Query Data:**
```ts
const { data, error } = await supabase
  .from('posts')
  .select('*')
  .eq('published', true)
  .order('created_at', { ascending: false })
```

**Insert Data:**
```ts
const { data, error } = await supabase
  .from('posts')
  .insert({ title: 'New Post', user_id: userId })
  .select()
```

**RLS Policy:**
```sql
CREATE POLICY "Users can view own posts"
ON public.posts FOR SELECT
TO authenticated
USING ((select auth.uid()) = user_id);
```

## RLS Performance

RLS policies execute for every row. Poorly written policies can cause exponential query times. Follow these patterns for performant RLS.

### Always Wrap Functions in SELECT

**Bad - function called per row:**
```sql
CREATE POLICY "Users can view own posts" ON posts
FOR SELECT USING (auth.uid() = user_id);
```

**Good - function result cached:**
```sql
CREATE POLICY "Users can view own posts" ON posts
FOR SELECT USING ((SELECT auth.uid()) = user_id);
```

The `(SELECT ...)` wrapper lets Postgres cache the function result instead of recalculating for every row.

### Index Columns Used in Policies

Always create indexes on columns referenced in RLS policies:

```sql
-- If your policy checks user_id
CREATE INDEX idx_posts_user_id ON posts(user_id);

-- If your policy checks organization_id
CREATE INDEX idx_posts_org_id ON posts(organization_id);
```

### Never Pass Row Data to Functions

**Bad - function called per row with row data:**
```sql
CREATE POLICY "check_access" ON documents
FOR SELECT USING (check_user_access(id, user_id));  -- SLOW!
```

**Good - use direct comparisons:**
```sql
CREATE POLICY "check_access" ON documents
FOR SELECT USING ((SELECT auth.uid()) = user_id);
```

Functions receiving row data cannot be cached and execute once per row.

### Avoid Subqueries When Possible

Subqueries in policies execute for every row:

**Slow - subquery per row:**
```sql
CREATE POLICY "org_members" ON documents
FOR SELECT USING (
  EXISTS (
    SELECT 1 FROM org_members
    WHERE org_id = documents.org_id
    AND user_id = (SELECT auth.uid())
  )
);
```

**Faster - use SECURITY DEFINER function:**
```sql
-- Create a function that bypasses RLS
CREATE OR REPLACE FUNCTION get_user_org_ids()
RETURNS SETOF uuid AS $$
  SELECT org_id FROM org_members WHERE user_id = auth.uid()
$$ LANGUAGE sql SECURITY DEFINER STABLE;

-- Use it in policy
CREATE POLICY "org_members" ON documents
FOR SELECT USING (org_id IN (SELECT get_user_org_ids()));
```

### Consider Denormalization

For high-traffic tables, denormalize the user_id directly onto the row instead of joining:

```sql
-- Instead of joining through organizations
-- Add user_id directly to the documents table
ALTER TABLE documents ADD COLUMN owner_user_id uuid;

-- Simple, fast policy
CREATE POLICY "owner_access" ON documents
FOR SELECT USING ((SELECT auth.uid()) = owner_user_id);
```

### Test RLS Performance

Enable query analysis to debug slow policies:

```sql
-- Enable EXPLAIN for PostgREST
ALTER ROLE authenticator SET pgrst.db_plan_enabled TO true;
NOTIFY pgrst, 'reload config';
```

Then use `.explain()` in your Supabase client to see query plans.

### ACL Storage Strategy

For complex permissions (sharing with users/groups), where you store the ACL matters:

**Option 1: ACL in column (fastest)**
```sql
-- Store permitted user IDs directly on the row
ALTER TABLE documents ADD COLUMN read_access uuid[] DEFAULT '{}';
ALTER TABLE documents ADD COLUMN write_access uuid[] DEFAULT '{}';

-- Add GIN index for array operations
CREATE INDEX idx_docs_read ON documents USING GIN(read_access);

-- Fast policy using array overlap
CREATE POLICY "shared_access" ON documents
FOR SELECT USING (
  read_access && ARRAY[(SELECT auth.uid())]
);
```

**Option 2: ACL in separate table (slower but normalized)**
```sql
-- Separate permissions table
CREATE TABLE document_permissions (
  document_id uuid REFERENCES documents(id),
  user_id uuid,
  role text  -- 'read', 'write', 'owner'
);

-- Policy with subquery (slower)
CREATE POLICY "shared_access" ON documents
FOR SELECT USING (
  EXISTS (
    SELECT 1 FROM document_permissions
    WHERE document_id = documents.id
    AND user_id = (SELECT auth.uid())
  )
);
```

| Approach | Speed | Trade-off |
|----------|-------|-----------|
| ACL in column | ~1ms | Denormalized, wide rows if many permissions |
| ACL in table | ~20ms+ | Normalized, but slower JOINs |

**Recommendation:** For most apps, use column-based ACL. Only use a separate table if you need complex permission hierarchies or audit trails.

### Avoid OR with Boolean Columns

Adding `OR public = TRUE` to policies forces sequential scans:

```sql
-- Bad - forces sequential scan
CREATE POLICY "public_or_owner" ON items
FOR SELECT USING (
  public = TRUE OR (SELECT auth.uid()) = user_id
);

-- Better - use a "public" group ID instead
-- Add the public group ID to the ACL array
```

### Performance Checklist

| Pattern | Performance |
|---------|-------------|
| `(SELECT auth.uid()) = user_id` | Fast - cached |
| `auth.uid() = user_id` | Slow - per row |
| Direct column comparison | Fast |
| Subquery per row | Slow |
| SECURITY DEFINER function | Fast |
| Function with row data param | Slow |
| Indexed columns in policy | Fast |
| Non-indexed columns | Slow |
| ACL in column with GIN index | Fast |
| ACL in separate table | Slow |
| `OR public = TRUE` | Slow - sequential scan |

**Get User:**
```ts
const { data: { user } } = await supabase.auth.getUser()
```

## Reference Files

- [rls.md](rls.md) - Row Level Security patterns
- [auth.md](auth.md) - Authentication setup
- [edge-functions.md](edge-functions.md) - Edge Functions guide

## Critical Rules

**NEVER use deprecated cookie patterns:**
```ts
// WRONG - will break the app
cookies: { get(name), set(name, value), remove(name) }

// CORRECT - always use this
cookies: { getAll(), setAll(cookiesToSet) }
```

**NEVER import from auth-helpers-nextjs** - it's deprecated.

## Using an ORM with Supabase

You have three options for querying your Supabase database:

### Option 1: Supabase JS Client (Recommended for most cases)

Use the built-in `@supabase/supabase-js` client shown above. This is the simplest option and integrates well with Supabase Auth and RLS.

### Option 2: Prisma ORM

If you prefer Prisma's schema-based approach and migration tooling, you need to install an adapter.

**Install packages:**
```bash
pnpm add @prisma/adapter-pg pg
pnpm add -D prisma @types/pg
```

**Get connection strings from Supabase:**
1. Go to Supabase Dashboard → Connect → ORMs → Prisma
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

### Option 3: Drizzle ORM

If you prefer Drizzle's SQL-like syntax and lighter footprint.

**Install packages:**
```bash
pnpm add drizzle-orm postgres
pnpm add -D drizzle-kit
```

**Get connection string from Supabase:**
1. Go to Supabase Dashboard → Connect → ORMs
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

### Which option should I choose?

| Option | Best for |
|--------|----------|
| **Supabase JS** | Most projects - simple queries, using Supabase Auth/RLS |
| **Prisma** | Teams wanting visual Studio browser, robust migrations, higher-level abstraction |
| **Drizzle** | Developers comfortable with SQL, edge/serverless deployments, smaller bundle size |

## Realtime Subscriptions

Supabase includes built-in real-time sync. Use this when multiple users need to see updates instantly (collaborative features, live dashboards, chat).

**Subscribe to table changes:**
```ts
import { createClient } from '@/lib/supabase/client'

const supabase = createClient()

// Subscribe to all changes on a table
const channel = supabase
  .channel('contacts-changes')
  .on(
    'postgres_changes',
    { event: '*', schema: 'public', table: 'contacts' },
    (payload) => {
      console.log('Change:', payload.eventType, payload.new)
      // Update your local state/UI here
    }
  )
  .subscribe()

// Cleanup when component unmounts
return () => {
  supabase.removeChannel(channel)
}
```

**Filter by specific events:**
```ts
// Only inserts
.on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'messages' }, handler)

// Only updates
.on('postgres_changes', { event: 'UPDATE', schema: 'public', table: 'contacts' }, handler)

// Only deletes
.on('postgres_changes', { event: 'DELETE', schema: 'public', table: 'contacts' }, handler)
```

**Filter by row:**
```ts
// Only changes to a specific organization's contacts
.on(
  'postgres_changes',
  {
    event: '*',
    schema: 'public',
    table: 'contacts',
    filter: `organization_id=eq.${orgId}`
  },
  handler
)
```

**React hook pattern:**
```tsx
import { useEffect, useState } from 'react'
import { createClient } from '@/lib/supabase/client'

function useRealtimeContacts(orgId: string) {
  const [contacts, setContacts] = useState<Contact[]>([])
  const supabase = createClient()

  useEffect(() => {
    // Initial fetch
    supabase
      .from('contacts')
      .select('*')
      .eq('organization_id', orgId)
      .then(({ data }) => setContacts(data || []))

    // Subscribe to changes
    const channel = supabase
      .channel(`contacts-${orgId}`)
      .on(
        'postgres_changes',
        {
          event: '*',
          schema: 'public',
          table: 'contacts',
          filter: `organization_id=eq.${orgId}`
        },
        (payload) => {
          if (payload.eventType === 'INSERT') {
            setContacts(prev => [...prev, payload.new as Contact])
          } else if (payload.eventType === 'UPDATE') {
            setContacts(prev =>
              prev.map(c => c.id === payload.new.id ? payload.new as Contact : c)
            )
          } else if (payload.eventType === 'DELETE') {
            setContacts(prev => prev.filter(c => c.id !== payload.old.id))
          }
        }
      )
      .subscribe()

    return () => {
      supabase.removeChannel(channel)
    }
  }, [orgId])

  return contacts
}
```

**Enable Realtime on a table (required):**
```sql
-- In Supabase SQL Editor or migration
ALTER PUBLICATION supabase_realtime ADD TABLE contacts;
```

Or enable via Dashboard: Table Editor → Select table → Enable Realtime

### When to use Realtime

| Use case | Realtime needed? |
|----------|------------------|
| Multiple users editing same data | Yes |
| Live dashboards/analytics | Yes |
| Chat/messaging features | Yes |
| Single user CRUD | No - standard queries are fine |
| Data that rarely changes | No |

### Realtime vs Electric SQL

If you need real-time sync, Supabase Realtime is the simplest choice when using Supabase. You don't need Electric SQL - that's for self-hosted Postgres without Supabase.

## PrivateLink (Enterprise)

For compliance requirements (healthcare, finance, public sector) or to minimize attack surface, Supabase offers PrivateLink - private database connections through AWS networks.

### What It Does

- Database traffic never leaves private AWS networks
- Your Supabase database appears to exist within your own VPC
- Lower latency than public connections (more direct routing)
- Can disable public database access entirely once configured

### Requirements

| Requirement | Details |
|-------------|---------|
| Plan | Team or Enterprise only |
| Cloud | AWS environments only |
| Region | Same AWS region as your Supabase project |
| Scope | Database connections only (not API, Auth, Storage, or Realtime) |

### When to Use

- Compliance requires private network connectivity
- Want to eliminate public database endpoints
- Workloads already running on AWS
- Need to meet SOC 2, HIPAA, or similar requirements

### When NOT to Use

- Free or Pro plan (not available)
- Non-AWS hosting (Vercel serverless, Cloudflare, etc.)
- Need private access to Auth, Storage, or Realtime (not supported yet)

### Setup Overview

1. **Add AWS Account ID** - In Supabase Dashboard > Project Settings > Infrastructure
2. **Accept Resource Share** - In AWS RAM (Resource Access Manager)
3. **Create VPC Endpoint** - In AWS VPC console, create endpoint for the Supabase service
4. **Configure Security Groups** - Allow TCP port 5432 inbound
5. **Update Connection String** - Use the private endpoint hostname
6. **Test Connection** - Verify connectivity from your VPC

### Connection String

After setup, your connection string changes from:
```
postgresql://postgres.[ref]:[password]@aws-0-[region].pooler.supabase.com:5432/postgres
```

To the private endpoint:
```
postgresql://postgres.[ref]:[password]@vpce-xxx.supabase.com:5432/postgres
```

### Limitations

- **Database only** - API calls, Auth, Storage, and Realtime still use public endpoints
- **Same region** - Your VPC must be in the same AWS region as your Supabase project
- **AWS only** - Not available for GCP, Azure, or other cloud providers

For most projects, the standard public connection with SSL is sufficient. PrivateLink is for organizations with strict compliance requirements.

## Pre-Request Hooks

Run custom validation before every Data API request. Useful for rate limiting, API key validation, or blocking access to specific tables.

### Setup

```sql
-- Create a function that runs before every request
CREATE OR REPLACE FUNCTION public.check_request()
RETURNS void AS $$
DECLARE
  req_path text := current_setting('request.path', true);
  req_headers json := current_setting('request.headers', true)::json;
BEGIN
  -- Example: Require custom API key header
  IF req_headers->>'x-api-key' IS NULL THEN
    RAISE EXCEPTION 'API key required'
      USING HINT = 'Include x-api-key header';
  END IF;

  -- Example: Block access to admin tables from API
  IF req_path LIKE '%admin_%' THEN
    RAISE EXCEPTION 'Access denied'
      USING HINT = 'Admin tables not accessible via API';
  END IF;
END;
$$ LANGUAGE plpgsql;

-- Enable the pre-request hook
ALTER ROLE authenticator SET pgrst.db_pre_request = 'public.check_request';
```

### Available Request Context

Inside your function, access request info with `current_setting()`:

| Setting | What It Contains |
|---------|------------------|
| `request.path` | API path (e.g., `/rest/v1/users`) |
| `request.headers` | JSON object of all headers |
| `request.method` | HTTP method (GET, POST, etc.) |
| `request.jwt.claims` | JWT payload (user info) |

### Use Cases

| Use Case | Implementation |
|----------|----------------|
| Rate limiting | Check counter in a rate_limits table |
| API key validation | Verify header against api_keys table |
| Block direct table access | Check path and reject certain patterns |
| Payment/quota checks | Query user's subscription status |
| IP allowlisting | Check request.headers->>'x-forwarded-for' |

### Limitations

- **Data API only** - Does not apply to Realtime or Storage
- For Realtime/Storage, add validation inside RLS policies instead

### When to Use

Most apps don't need pre-request hooks. Use them when:
- You need rate limiting at the database level
- You want to validate custom API keys
- You need to block certain tables from direct API access
- RLS alone isn't sufficient for your security requirements

## Security Advisor

Supabase includes a built-in Security Advisor that scans your project for misconfigurations.

**Access it:** Dashboard > Project Settings > Security Advisor

**What it checks:**
- Tables without RLS enabled
- Overly permissive RLS policies
- Exposed sensitive columns
- Missing indexes on foreign keys
- Other common security issues

**Run before production** - The Security Advisor catches issues that are easy to miss during development.

**AI Policy Assistant:** The dashboard also includes an AI assistant that can generate RLS policies from plain-language descriptions. Useful for complex policies.

## Tips

- Always enable RLS on new tables (enabled by default since 2025)
- Use `(select auth.uid())` in policies for better performance
- Run `npx supabase gen types typescript` after schema changes
- Use snake_case for database columns
- Free Supabase accounts allow up to 2 projects
- Enable Realtime per-table (not enabled by default)
- Run Security Advisor before deploying to production
