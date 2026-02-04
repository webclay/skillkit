# Database Security Remediation Guide

How to fix common security issues found during audits.

## Issue: Frontend Direct Database Access

### Problem
Frontend components directly calling Supabase/database, bypassing backend security.

### Solution

1. **Create API routes for all data operations:**

```typescript
// app/api/posts/route.ts
import { auth } from '@/lib/auth';
import { db } from '@/lib/db';

export async function GET(request: Request) {
  const session = await auth.api.getSession({ headers: request.headers });
  if (!session) {
    return Response.json({ error: 'Unauthorized' }, { status: 403 });
  }

  const posts = await db.post.findMany({
    where: { userId: session.user.id }
  });

  return Response.json(posts);
}
```

2. **Update frontend to use API:**

```typescript
// Before (bad)
const { data } = await supabase.from('posts').select('*');

// After (good)
const response = await fetch('/api/posts');
const posts = await response.json();
```

3. **Remove browser client data methods:**

Only use browser Supabase client for auth:

```typescript
// lib/supabase-client.ts
// ONLY use for auth, not data!
export const supabase = createBrowserClient(url, anonKey);

// Usage: auth only
const { data: { user } } = await supabase.auth.getUser();
```

---

## Issue: RLS Policies as Primary Security

### Problem
Relying on RLS policies to handle authorization instead of backend checks.

### Solution

1. **Enable RLS with no permissive policies:**

```sql
-- Remove permissive policies
DROP POLICY IF EXISTS "Users can view own data" ON posts;
DROP POLICY IF EXISTS "Users can insert own data" ON posts;

-- Keep RLS enabled (blocks all access except service role)
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;
```

2. **Use service role in backend:**

```typescript
// lib/supabase-admin.ts (server only!)
import { createClient } from '@supabase/supabase-js';

export const supabaseAdmin = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!, // Service role bypasses RLS
  { auth: { persistSession: false } }
);
```

3. **Never import admin client in frontend code.**

---

## Issue: Missing Auth Checks

### Problem
API routes or server actions don't verify authentication.

### Solution

1. **Add auth check to every route:**

```typescript
// Template for all API routes
import { auth } from '@/lib/auth';

export async function GET(request: Request) {
  // ALWAYS FIRST
  const session = await auth.api.getSession({ headers: request.headers });
  if (!session) {
    return Response.json({ error: 'Unauthorized' }, { status: 403 });
  }

  // Rest of logic...
}
```

2. **Create auth helper for reuse:**

```typescript
// lib/require-auth.ts
import { auth } from '@/lib/auth';

export async function requireAuth(request: Request) {
  const session = await auth.api.getSession({ headers: request.headers });
  if (!session) {
    throw new Response(JSON.stringify({ error: 'Unauthorized' }), { status: 403 });
  }
  return session;
}

// Usage
export async function GET(request: Request) {
  const session = await requireAuth(request);
  // ...
}
```

3. **For server actions:**

```typescript
'use server';

import { auth } from '@/lib/auth';
import { headers } from 'next/headers';

export async function updateProfile(data: FormData) {
  const session = await auth.api.getSession({ headers: await headers() });
  if (!session) {
    return { error: 'Unauthorized' };
  }

  // Safe to proceed...
}
```

---

## Issue: Missing User ID Filtering

### Problem
Queries don't filter by userId, potentially exposing other users' data.

### Solution

1. **Always include userId in where clause:**

```typescript
// Before (bad)
const posts = await db.post.findMany({
  where: { published: true }
});

// After (good)
const posts = await db.post.findMany({
  where: {
    published: true,
    userId: session.user.id // Always filter!
  }
});
```

2. **For updates and deletes, verify ownership:**

```typescript
// Before (bad) - anyone can delete any post
await db.post.delete({ where: { id: postId } });

// After (good) - only owner can delete
const post = await db.post.findFirst({
  where: { id: postId, userId: session.user.id }
});
if (!post) {
  return Response.json({ error: 'Not found' }, { status: 404 });
}
await db.post.delete({ where: { id: postId } });
```

---

## Issue: Exposed Service Key

### Problem
`SUPABASE_SERVICE_ROLE_KEY` or `DATABASE_URL` in client-accessible code.

### Solution

1. **Move to server-only environment:**

```env
# .env.local
# These should NOT have NEXT_PUBLIC_ prefix
SUPABASE_SERVICE_ROLE_KEY=your-key
DATABASE_URL=your-url

# These are safe to expose
NEXT_PUBLIC_SUPABASE_URL=your-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
```

2. **Verify not in client bundle:**

```bash
# Build and search output
npm run build
grep -r "service_role" .next/
grep -r "your-database-url" .next/
```

3. **Rotate compromised keys immediately:**

If service key was exposed:
- Generate new key in Supabase dashboard
- Update server environment variables
- Redeploy server
- Old key is immediately invalid

---

## Issue: Late Auth Check

### Problem
Request is processed before checking authentication.

### Solution

Auth check must be FIRST:

```typescript
export async function POST(request: Request) {
  // Step 1: Auth check FIRST
  const session = await auth.api.getSession({ headers: request.headers });
  if (!session) {
    return Response.json({ error: 'Unauthorized' }, { status: 403 });
  }

  // Step 2: Only then parse body
  const body = await request.json();

  // Step 3: Then validate
  const validated = schema.parse(body);

  // Step 4: Then execute
  await db.post.create({ data: { ...validated, userId: session.user.id } });
}
```

---

## Secure API Route Template

Use this template for all new API routes:

```typescript
// app/api/[resource]/route.ts
import { auth } from '@/lib/auth';
import { db } from '@/lib/db';
import { z } from 'zod';

const createSchema = z.object({
  title: z.string().min(1).max(100),
});

export async function GET(request: Request) {
  // 1. Auth first
  const session = await auth.api.getSession({ headers: request.headers });
  if (!session) {
    return Response.json({ error: 'Unauthorized' }, { status: 403 });
  }

  // 2. Query with user filter
  const items = await db.item.findMany({
    where: { userId: session.user.id }
  });

  return Response.json(items);
}

export async function POST(request: Request) {
  // 1. Auth first
  const session = await auth.api.getSession({ headers: request.headers });
  if (!session) {
    return Response.json({ error: 'Unauthorized' }, { status: 403 });
  }

  // 2. Parse body
  const body = await request.json();

  // 3. Validate
  const parsed = createSchema.safeParse(body);
  if (!parsed.success) {
    return Response.json({ error: 'Invalid input' }, { status: 400 });
  }

  // 4. Create with user ID from session (not from body!)
  const item = await db.item.create({
    data: { ...parsed.data, userId: session.user.id }
  });

  return Response.json(item, { status: 201 });
}

export async function DELETE(request: Request) {
  // 1. Auth first
  const session = await auth.api.getSession({ headers: request.headers });
  if (!session) {
    return Response.json({ error: 'Unauthorized' }, { status: 403 });
  }

  // 2. Get ID from query
  const { searchParams } = new URL(request.url);
  const id = searchParams.get('id');
  if (!id) {
    return Response.json({ error: 'ID required' }, { status: 400 });
  }

  // 3. Verify ownership before delete
  const item = await db.item.findFirst({
    where: { id, userId: session.user.id }
  });
  if (!item) {
    return Response.json({ error: 'Not found' }, { status: 404 });
  }

  // 4. Delete
  await db.item.delete({ where: { id } });

  return Response.json({ success: true });
}
```

---

## Secure Server Action Template

```typescript
'use server';

import { auth } from '@/lib/auth';
import { db } from '@/lib/db';
import { headers } from 'next/headers';
import { z } from 'zod';

const updateSchema = z.object({
  id: z.string(),
  title: z.string().min(1).max(100),
});

export async function updateItem(formData: FormData) {
  // 1. Auth first
  const session = await auth.api.getSession({ headers: await headers() });
  if (!session) {
    return { error: 'Unauthorized' };
  }

  // 2. Validate input
  const parsed = updateSchema.safeParse({
    id: formData.get('id'),
    title: formData.get('title'),
  });
  if (!parsed.success) {
    return { error: 'Invalid input' };
  }

  // 3. Verify ownership
  const item = await db.item.findFirst({
    where: { id: parsed.data.id, userId: session.user.id }
  });
  if (!item) {
    return { error: 'Not found' };
  }

  // 4. Update
  await db.item.update({
    where: { id: parsed.data.id },
    data: { title: parsed.data.title }
  });

  return { success: true };
}
```
