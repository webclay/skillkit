---
name: supabase
description: Use this skill when building with Supabase for database operations, row-level security, auth, edge functions, or real-time subscriptions. Activate when the user mentions Supabase, Postgres RLS policies, Supabase Auth, Edge Functions, or Supabase Storage.
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

**Get User:**
```ts
const { data: { user } } = await supabase.auth.getUser()
```

## Critical Rules

**NEVER use deprecated cookie patterns:**
```ts
// WRONG - will break the app
cookies: { get(name), set(name, value), remove(name) }

// CORRECT - always use this
cookies: { getAll(), setAll(cookiesToSet) }
```

**NEVER import from auth-helpers-nextjs** - it's deprecated.

## Tips

- Always enable RLS on new tables (enabled by default since 2025)
- Use `(select auth.uid())` in policies for better performance
- Run `npx supabase gen types typescript` after schema changes
- Use snake_case for database columns
- Free Supabase accounts allow up to 2 projects
- Enable Realtime per-table (not enabled by default)
- Run Security Advisor before deploying to production

## Reference Files

Read these when working on specific topics:

- [rls-performance.md](rls-performance.md) - RLS optimization patterns, anti-patterns, ACL strategies, performance checklist
- [security-architecture.md](security-architecture.md) - Model 1 (RLS) vs Model 2 (Middleware), decision matrix, migration path, examples
- [orm-integration.md](orm-integration.md) - Prisma adapter setup, Drizzle setup, comparison table
- [realtime.md](realtime.md) - Subscribe to changes, filters, React hook pattern, when to use
- [advanced-features.md](advanced-features.md) - PrivateLink (enterprise), Pre-Request Hooks, Security Advisor
- [rls.md](rls.md) - Row Level Security patterns
- [auth.md](auth.md) - Authentication setup
- [edge-functions.md](edge-functions.md) - Edge Functions guide
