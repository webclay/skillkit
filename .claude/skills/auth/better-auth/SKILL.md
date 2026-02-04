---
name: better-auth
description: Better Auth authentication patterns and implementation. Trigger words - login, signup, sign up, sign in, logout, auth, authentication, OAuth, social login, password reset, forgot password, session, protected routes, user account
---

# Better Auth

Adds user authentication to your app - email & password signup/login, social login (Google, GitHub), session management, protected routes, and password reset.

## When to Use This Skill

- User asks to add user login, signup, or authentication
- package.json contains "better-auth"
- User mentions OAuth, social login, Google/GitHub login
- User mentions password reset, email verification, 2FA
- User asks about sessions, protected routes, or auth middleware

## Security: Server-Side Authentication Required

**Authentication MUST be validated server-side.** This applies to ALL frameworks (Next.js, TanStack Start, Remix, etc.).

- **Server-side auth = security** - Blocks requests before data is accessed
- **Client-side auth = UX only** - Shows/hides UI elements, but never protects data

**Never rely on client-side session checks alone.** A client-side `useSession()` + redirect pattern is a security vulnerability - the page content loads before the redirect happens.

## Framework-Specific Setup

Check your framework and follow the appropriate guide:

| Framework | Guide |
|-----------|-------|
| TanStack Start | [tanstack-start.md](tanstack-start.md) - **Required reading for TanStack projects** |
| Next.js | Continue below |
| Convex | [convex.md](convex.md) - Different setup required |

## Important: Convex Integration

**If the project uses Convex as the database, follow [convex.md](convex.md) instead of the standard setup below.** The Convex integration requires:
- `@convex-dev/better-auth` package (not just `better-auth`)
- Different configuration files and providers
- Auth handled via Convex HTTP routes, not standard API routes

Check for `"convex"` in package.json to determine if Convex integration is needed.

## Instructions (Next.js with Prisma/Drizzle)

### Step 1: Install Better Auth

```bash
pnpm add better-auth
```

### Step 2: Create Server Auth Config

Create `lib/auth.ts`:

```ts
import { betterAuth } from "better-auth";
import { prismaAdapter } from "better-auth/adapters/prisma";
import { db } from "./db";

export const auth = betterAuth({
  database: prismaAdapter(db, { provider: "postgresql" }),
  emailAndPassword: { enabled: true },
  session: {
    expiresIn: 60 * 60 * 24 * 7, // 7 days
  },
});
```

### Step 3: Create API Route

Create `app/api/auth/[...all]/route.ts`:

```ts
import { auth } from "@/lib/auth";
import { toNextJsHandler } from "better-auth/next-js";

export const { POST, GET } = toNextJsHandler(auth);
```

### Step 4: Create Client

Create `lib/auth-client.ts`:

```ts
import { createAuthClient } from "better-auth/react";

export const authClient = createAuthClient({
  baseURL: process.env.NEXT_PUBLIC_APP_URL || "http://localhost:3000",
});

export const { signIn, signUp, signOut, useSession } = authClient;
```

### Step 5: Set Environment Variables

```env
BETTER_AUTH_SECRET=<random-32-char-secret>
BETTER_AUTH_URL=http://localhost:3000
DATABASE_URL=postgresql://...
```

### Step 6: Add Database Tables

See [database.md](database.md) for Prisma schema, or use CLI:
```bash
npx better-auth generate
npx better-auth migrate
```

### Step 7: Protect Routes (Server-Side)

Create `middleware.ts` in project root:

```ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";
import { auth } from "@/lib/auth";

export async function middleware(request: NextRequest) {
  const session = await auth.api.getSession({
    headers: request.headers,
  });

  const isAuthPage = request.nextUrl.pathname.startsWith("/sign-in") ||
                     request.nextUrl.pathname.startsWith("/sign-up");
  const isProtectedPage = request.nextUrl.pathname.startsWith("/dashboard");

  // Redirect authenticated users away from auth pages
  if (session && isAuthPage) {
    return NextResponse.redirect(new URL("/dashboard", request.url));
  }

  // Redirect unauthenticated users to login
  if (!session && isProtectedPage) {
    return NextResponse.redirect(new URL("/sign-in", request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ["/dashboard/:path*", "/sign-in", "/sign-up"],
};
```

## Examples

**Sign Up:**
```tsx
await signUp.email({ email, password, name, callbackURL: "/dashboard" });
```

**Sign In:**
```tsx
await signIn.email({ email, password, callbackURL: "/dashboard" });
```

**Check Session (client-side for UI only):**
```tsx
const { data: session, isPending } = useSession();
if (!session) return <div>Not logged in</div>;
return <div>Welcome, {session.user.name}!</div>;
```

**Sign Out:**
```tsx
await signOut({ fetchOptions: { onSuccess: () => router.push("/") } });
```

**Social Login:**
```tsx
await signIn.social({ provider: "google", callbackURL: "/dashboard" });
```

**Protected API Route (always validate server-side):**
```ts
const session = await auth.api.getSession({ headers: request.headers });
if (!session) return new Response("Unauthorized", { status: 401 });
```

## Reference Files

- [tanstack-start.md](tanstack-start.md) - **TanStack Start complete setup** (server-side auth patterns)
- [trpc.md](trpc.md) - **tRPC authentication** (protected procedures)
- [setup.md](setup.md) - Full installation and configuration
- [forms.md](forms.md) - Complete form examples with error handling
- [oauth.md](oauth.md) - Social login providers setup
- [middleware.md](middleware.md) - Route protection patterns (all frameworks)
- [database.md](database.md) - Prisma schema and database setup
- [convex.md](convex.md) - Convex integration (use when Convex is your database)

## Tips

- Always filter queries by `userId`: `where: { userId: session.user.id }`
- Check https://www.better-auth.com for latest features
- Server-side validation is required - client-side checks are for UX only

## Critical: Schema ID Requirements

Better Auth generates its own **string IDs** (not UUIDs). Do **NOT** use `@db.Uuid` or `@default(uuid())` on Better Auth tables.

**Correct:**
```prisma
model user {
  id            String    @id  // No @default(uuid()) or @db.Uuid
  email         String    @unique
  // ...
}
```

**Wrong (causes "invalid input syntax for type uuid" errors):**
```prisma
model user {
  id    String @id @default(uuid()) @db.Uuid  // DON'T DO THIS
  // ...
}
```

For your own application tables (quizzes, templates, etc.), you can use `@default(uuid())` but still omit `@db.Uuid`:

```prisma
model quizzes {
  id      String @id @default(uuid())  // OK for your own tables
  user_id String
  // ...
}
```

## How to Verify

### Quick Checks
- `curl http://localhost:3000/api/auth/session` - Auth endpoint responds
- Check browser Network tab for auth API calls (should return 200)
- Database has user/session tables created

### Manual Verification
- Sign up with a test email and password - should succeed
- Sign in with those credentials - should redirect to callbackURL
- Check `useSession()` returns the logged-in user data
- Sign out and confirm session is cleared

### Common Issues
- "BETTER_AUTH_SECRET required": Add secret to .env file
- "Database adapter error": Check DATABASE_URL and run migrations
- Session not persisting: Verify cookies are being set (check browser DevTools)
- OAuth redirect fails: Check callback URLs match in provider settings
- "invalid input syntax for type uuid": Remove `@db.Uuid` and `@default(uuid())` from Better Auth tables (user, session, account, verification) - Better Auth generates its own string IDs
