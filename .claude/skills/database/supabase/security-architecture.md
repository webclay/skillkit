# Security Architecture: Choosing Your Approach

Supabase projects can use two different security models. Choose based on your architecture.

## Model 1: Database-Layer Security (RLS Policies)

**How it works:**
- Frontend code directly queries the database using the anon key
- Row Level Security (RLS) policies enforce permissions at the database level
- Every query is automatically filtered by RLS rules
- No backend API layer required

**When to use:**
- Building a traditional Supabase app
- Using `createBrowserClient()` in components
- Want to minimize backend code
- Small to medium apps with simple permissions
- Client-side rendering (CSR) or static sites

**Setup:**
```ts
// Frontend component directly queries DB
import { createClient } from '@/lib/supabase/client'

const supabase = createClient() // Uses NEXT_PUBLIC_SUPABASE_ANON_KEY
const { data } = await supabase
  .from('posts')
  .select('*') // RLS policy controls what's returned
```

**Required:**
- RLS enabled on all tables
- RLS policies implemented for each table
- Anon key exposed to frontend

**Pros:**
- Simple setup for basic apps
- No API routes to maintain
- Supabase handles auth + data

**Cons:**
- Complex permissions require complex policies
- Can't change logic without app redeployment
- RLS performance can be tricky at scale
- Limited ability to cut off access in emergencies

## Model 2: Application-Layer Security (Middleware Auth)

**How it works:**
- Backend API routes/server actions query the database using service_role key
- Middleware checks authentication before any database access
- Authorization logic in your backend code (route handlers, server actions)
- Frontend never directly accesses the database

**When to use:**
- Server-side rendering (Next.js App Router, TanStack Start, Remix, SvelteKit)
- Complex business logic or permissions
- Need to update security logic without redeploying clients (critical for mobile apps)
- Want full control over data access
- Medium to large apps

**Setup:**
```ts
// API route with middleware auth
export async function GET(request: Request) {
  // Auth check FIRST
  const user = await requireAuthenticatedUser(request)
  if (!user) return Response.json({ error: 'Unauthorized' }, { status: 403 })

  // Then query with service role (bypasses RLS)
  const posts = await db.posts.findMany({
    where: { userId: user.id } // Authorization in code
  })

  return Response.json(posts)
}
```

**Required:**
- Auth middleware on all API routes/server actions
- Service role key (server-side only, never exposed)
- Authorization logic in backend code
- RLS enabled but no policies needed (defense in depth)

**Pros:**
- Full control over access logic
- Can update security without redeploying clients
- Can cut access immediately if issues arise
- Better for complex permissions
- Easier to debug and test

**Cons:**
- More backend code to write
- Need to implement API routes or server actions
- Slightly more complex setup

## Understanding RLS Warnings

If you see "RLS enabled but no policies exist", the meaning depends on your security model:

| Your Architecture | Is this OK? | What to do |
|-------------------|-------------|------------|
| Frontend queries DB directly (Model 1) | No - security risk | Implement RLS policies immediately |
| Backend API routes + middleware (Model 2) | Yes - expected behavior | Ignore warnings, this is normal |
| Using service role key only | Yes - expected behavior | Keep RLS enabled as defense in depth |
| Exposing anon key to frontend | No - security risk | Either add RLS policies or switch to Model 2 |

**Why enable RLS without policies?**

Even if you use Model 2 (application-layer security), keep RLS enabled with no policies:
- **Defense in depth:** If your service role key leaks, RLS locks down everything
- **Future-proofing:** Easy to add policies later if needed
- **Industry best practice:** RLS on by default is safer

The warnings are informational, not security risks, when using Model 2.

## Decision Matrix

**Choose Model 1 (RLS Policies) if:**
- Simple CRUD app with basic permissions
- Want minimal backend code
- Using Supabase Auth
- Permissions are user-scoped (users own their data)

**Choose Model 2 (Middleware Auth) if:**
- Server-rendered app (Next.js App Router, TanStack Start)
- Complex business logic or multi-tenant permissions
- Need to update security logic without redeployments
- Mobile app (can't force users to update)
- Want full control and auditability
- Working with a team that prefers backend authorization

**Can't decide?** Start with Model 2 (application-layer security) if you're using a server-rendered framework. It's more flexible and scales better for complex apps. Use Model 1 (RLS policies) only if you're building a simple app with straightforward permissions and want to minimize backend code.

## Migration Path

**Started with Model 1, need to switch to Model 2?**
1. Create API routes for all data operations
2. Move database queries from frontend to backend
3. Add auth middleware to all routes
4. Keep RLS enabled but policies become optional
5. Remove `createBrowserClient()` usage from components

**Started with Model 2, want to add RLS policies?**
1. Keep existing middleware auth (primary security)
2. Add RLS policies for defense in depth
3. Now you have dual-layer security (belt and suspenders)

## Example: Model 1 (RLS-Based Security)

**Supabase RLS Policy:**
```sql
-- Users can only see their own posts
CREATE POLICY "users_own_posts"
ON posts FOR SELECT
TO authenticated
USING ((SELECT auth.uid()) = user_id);

-- Users can only insert their own posts
CREATE POLICY "users_insert_posts"
ON posts FOR INSERT
TO authenticated
WITH CHECK ((SELECT auth.uid()) = user_id);
```

**Frontend Query:**
```ts
// Frontend component - safe because RLS filters
const { data } = await supabase
  .from('posts')
  .select('*') // RLS automatically filters to user's posts
```

## Example: Model 2 (Application-Layer Security)

**API Route:**
```ts
// app/api/posts/route.ts
import { requireAuthenticatedUser } from '@/lib/auth-middleware'
import { db } from '@/lib/db'

export async function GET(request: Request) {
  // Auth check first
  const user = await requireAuthenticatedUser(request)
  if (!user) {
    return Response.json({ error: 'Unauthorized' }, { status: 403 })
  }

  // Authorization in code - filter by user
  const posts = await db.post.findMany({
    where: { userId: user.id }
  })

  return Response.json(posts)
}
```

**Frontend Query:**
```ts
// Frontend component - calls API, never touches DB directly
const response = await fetch('/api/posts')
const posts = await response.json()
```

**Database Setup:**
```sql
-- RLS enabled but no policies (defense in depth)
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;
-- No CREATE POLICY statements needed
```

**Get User:**
```ts
const { data: { user } } = await supabase.auth.getUser()
```
