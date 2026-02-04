# Database Security Anti-Patterns

Code patterns that indicate security vulnerabilities. Use these to search the codebase during audits.

## Frontend Database Access (Critical)

### Supabase Client in Components

**Bad - Direct database access from client:**
```typescript
// components/PostList.tsx - BAD!
'use client';
import { createBrowserClient } from '@supabase/ssr';

export function PostList() {
  const supabase = createBrowserClient(...);

  useEffect(() => {
    // This bypasses your backend entirely!
    supabase.from('posts').select('*').then(...)
  }, []);
}
```

**Detection:**
```bash
grep -rn "\.from\(" --include="*.tsx" components/
grep -rn "createBrowserClient" --include="*.tsx" components/
```

### Mutations from Frontend

**Bad - Insert/Update/Delete from client:**
```typescript
// BAD - Anyone can insert data!
const { error } = await supabase
  .from('posts')
  .insert({ title, content, user_id: userId });
```

**Good - Through API route:**
```typescript
// app/api/posts/route.ts
export async function POST(request: Request) {
  const session = await auth.api.getSession({ headers: request.headers });
  if (!session) return new Response('Unauthorized', { status: 403 });

  // Now safe to insert with server's service role
  await db.post.create({ data: { ...body, userId: session.user.id } });
}
```

---

## RLS as Primary Security (Critical)

### Permissive Select Policies

**Bad - Relying on RLS for authorization:**
```sql
-- This is NOT secure as primary defense!
CREATE POLICY "Users can view own posts"
ON posts FOR SELECT
USING (auth.uid() = user_id);
```

**Why it's bad:**
- If anon key is compromised, attacker can impersonate users
- RLS policies can have bugs
- No audit trail
- Can't revoke access quickly

**Good - RLS as backup only:**
```sql
-- Enable RLS with NO policies (blocks everything)
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

-- Only service role can access
-- All access goes through backend API
```

### Permissive Insert/Update/Delete

**Critically Bad:**
```sql
-- NEVER DO THIS
CREATE POLICY "Anyone can insert"
ON posts FOR INSERT
WITH CHECK (true);

CREATE POLICY "Users can update own"
ON posts FOR UPDATE
USING (auth.uid() = user_id);
```

**Detection:**
```bash
grep -rn "CREATE POLICY" migrations/ supabase/
grep -rn "WITH CHECK (true)" migrations/
```

---

## Missing Auth Checks (Critical)

### API Route Without Auth

**Bad:**
```typescript
// app/api/users/route.ts - BAD!
export async function GET() {
  // No auth check!
  const users = await db.user.findMany();
  return Response.json(users);
}
```

**Good:**
```typescript
export async function GET(request: Request) {
  const session = await auth.api.getSession({ headers: request.headers });
  if (!session) {
    return new Response(JSON.stringify({ error: 'Unauthorized' }), {
      status: 403,
      headers: { 'Content-Type': 'application/json' }
    });
  }

  // Filter by user
  const data = await db.item.findMany({
    where: { userId: session.user.id }
  });
  return Response.json(data);
}
```

### Server Action Without Auth

**Bad:**
```typescript
'use server';

export async function deletePost(postId: string) {
  // No auth check!
  await db.post.delete({ where: { id: postId } });
}
```

**Good:**
```typescript
'use server';

export async function deletePost(postId: string) {
  const session = await auth.api.getSession({ headers: await headers() });
  if (!session) {
    throw new Error('Unauthorized');
  }

  // Verify ownership
  const post = await db.post.findUnique({
    where: { id: postId, userId: session.user.id }
  });
  if (!post) {
    throw new Error('Not found');
  }

  await db.post.delete({ where: { id: postId } });
}
```

---

## Exposed Secrets (Critical)

### Service Key in Client Code

**Bad:**
```typescript
// lib/supabase-client.ts - EXPOSED TO BROWSER!
const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_SERVICE_KEY! // NEVER DO THIS
);
```

**Detection:**
```bash
grep -rn "SERVICE" --include="*.ts" --include="*.tsx" .
grep -rn "NEXT_PUBLIC_.*SERVICE" .env*
grep -rn "NEXT_PUBLIC_.*KEY" .env* | grep -v ANON
```

### Database URL Exposed

**Bad:**
```typescript
// Anywhere in client-accessible code
const db = new PrismaClient({
  datasources: { db: { url: process.env.NEXT_PUBLIC_DATABASE_URL } }
});
```

---

## Missing User Filtering (High)

### Query Without userId Filter

**Bad:**
```typescript
// Even with auth, this returns ALL posts
const posts = await db.post.findMany({
  where: { published: true }
});
```

**Good:**
```typescript
const posts = await db.post.findMany({
  where: {
    published: true,
    userId: session.user.id // Always filter!
  }
});
```

### Delete Without Ownership Check

**Bad:**
```typescript
// Anyone with a valid session can delete any post!
await db.post.delete({ where: { id: postId } });
```

**Good:**
```typescript
// First verify ownership
const post = await db.post.findFirst({
  where: { id: postId, userId: session.user.id }
});
if (!post) {
  return new Response('Not found', { status: 404 });
}
await db.post.delete({ where: { id: postId } });
```

---

## Token Validation Issues (Medium)

### Not Checking Token Validity

**Bad:**
```typescript
// Just checking if token exists
const token = request.headers.get('Authorization');
if (!token) return unauthorized();
// Token could be expired or forged!
```

**Good:**
```typescript
const session = await auth.api.getSession({ headers: request.headers });
// Auth library validates token, checks expiration, etc.
if (!session) {
  return new Response('Unauthorized', { status: 403 });
}
```

### Late Auth Check

**Bad:**
```typescript
export async function POST(request: Request) {
  const body = await request.json();
  const validated = schema.parse(body); // Processing before auth!

  const session = await auth.api.getSession({ headers: request.headers });
  if (!session) return new Response('Unauthorized', { status: 403 });
}
```

**Good:**
```typescript
export async function POST(request: Request) {
  // Auth check FIRST
  const session = await auth.api.getSession({ headers: request.headers });
  if (!session) {
    return new Response('Unauthorized', { status: 403 });
  }

  // Then process
  const body = await request.json();
  const validated = schema.parse(body);
}
```

---

## Detection Commands Summary

```bash
# Frontend database access
grep -rn "\.from\(" --include="*.tsx" components/ app/
grep -rn "createBrowserClient" --include="*.tsx" components/
grep -rn "\.insert\(" --include="*.tsx" components/
grep -rn "\.update\(" --include="*.tsx" components/
grep -rn "\.delete\(" --include="*.tsx" components/

# RLS policies
grep -rn "CREATE POLICY" migrations/ supabase/
grep -rn "WITH CHECK (true)" migrations/

# Missing auth (look for routes without auth.api.getSession)
find app/api -name "route.ts" -exec grep -L "getSession" {} \;

# Exposed secrets
grep -rn "SERVICE" --include="*.ts" --include="*.tsx" lib/ app/
grep -rn "NEXT_PUBLIC_.*KEY" .env* | grep -v ANON
grep -rn "DATABASE_URL" --include="*.ts" --include="*.tsx" components/
```
