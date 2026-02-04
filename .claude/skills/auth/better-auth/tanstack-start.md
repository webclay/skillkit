# Better Auth with TanStack Start

Complete guide for implementing secure, server-side authentication in TanStack Start applications.

## Security First

**All authentication MUST be validated server-side.** TanStack Start runs on both server and client, so you need to protect:

1. **Routes** - Block page access before content loads (using `beforeLoad`)
2. **Server Functions** - Protect data operations (using middleware)
3. **API Routes** - Secure all data endpoints

**Never use client-side `useSession()` + redirect as your only protection.** That pattern loads the page first, then redirects - a security vulnerability.

## Step 1: Install Dependencies

```bash
pnpm add better-auth
```

## Step 2: Server Auth Config

Create `src/lib/auth.ts`:

```ts
import { betterAuth } from "better-auth";
import { prismaAdapter } from "better-auth/adapters/prisma";
import { tanstackStartCookies } from "better-auth/tanstack-start";
import { db } from "./db";

export const auth = betterAuth({
  database: prismaAdapter(db, { provider: "postgresql" }),
  emailAndPassword: { enabled: true },
  session: {
    expiresIn: 60 * 60 * 24 * 7, // 7 days
  },
  plugins: [
    tanstackStartCookies(), // Required for TanStack Start cookie handling
  ],
});
```

**Important:** The `tanstackStartCookies` plugin is required for proper cookie handling in TanStack Start. Without it, sessions may not persist correctly.

## Step 3: Auth Client

Create `src/lib/auth-client.ts`:

```ts
import { createAuthClient } from "better-auth/react";

export const authClient = createAuthClient({
  baseURL: import.meta.env.VITE_APP_URL || "http://localhost:3000",
});

export const { signIn, signUp, signOut, useSession } = authClient;
```

## Step 4: API Route Handler

Create `src/routes/api/auth/$.ts`:

```ts
import { createFileRoute } from "@tanstack/react-router";
import { auth } from "@/lib/auth";

export const Route = createFileRoute("/api/auth/$")({
  server: {
    handlers: {
      GET: ({ request }) => auth.handler(request),
      POST: ({ request }) => auth.handler(request),
    },
  },
});
```

## Step 5: Auth Middleware (Critical)

Create `src/lib/auth-middleware.ts`:

```ts
import { redirect } from "@tanstack/react-router";
import { createMiddleware } from "@tanstack/react-start";
import { createServerFn } from "@tanstack/react-start";
import { getRequest } from "@tanstack/react-start/server";
import { auth } from "./auth";

// Server function to get session from request headers
export const getSessionFn = createServerFn({ method: "GET" }).handler(
  async () => {
    const request = getRequest();
    if (!request?.headers) return null;

    const session = await auth.api.getSession({
      headers: request.headers,
    });

    return session;
  }
);

// Middleware for protecting server functions
export const authMiddleware = createMiddleware({ type: "function" }).server(
  async ({ next }) => {
    const session = await getSessionFn();

    if (!session) {
      throw redirect({ to: "/sign-in" });
    }

    return next({
      context: { session },
    });
  }
);

// Helper for API routes
export async function requireAuth(request: Request) {
  const session = await auth.api.getSession({
    headers: request.headers,
  });

  if (!session) {
    throw new Error("Unauthorized");
  }

  return session;
}

// Helper to get session without throwing
export async function getSession(request: Request) {
  return auth.api.getSession({ headers: request.headers });
}
```

## Step 6: Protected Route Layout

Create `src/routes/_authenticated.tsx`:

```tsx
import { createFileRoute, Outlet, redirect } from "@tanstack/react-router";
import { getSessionFn } from "@/lib/auth-middleware";

export const Route = createFileRoute("/_authenticated")({
  beforeLoad: async () => {
    const session = await getSessionFn();

    if (!session) {
      throw redirect({ to: "/sign-in" });
    }

    return { session };
  },
  component: AuthenticatedLayout,
});

function AuthenticatedLayout() {
  const { session } = Route.useRouteContext();

  return (
    <div>
      <header>
        <span>Welcome, {session.user.name}</span>
      </header>
      <Outlet />
    </div>
  );
}
```

Now any route under `_authenticated/` is protected server-side.

## Step 7: Protected Pages

Create protected pages under the authenticated layout, e.g. `src/routes/_authenticated/dashboard.tsx`:

```tsx
import { createFileRoute } from "@tanstack/react-router";

export const Route = createFileRoute("/_authenticated/dashboard")({
  component: DashboardPage,
});

function DashboardPage() {
  const { session } = Route.useRouteContext();

  return (
    <div>
      <h1>Dashboard</h1>
      <p>User ID: {session.user.id}</p>
    </div>
  );
}
```

## Step 8: Protected Server Functions

Use the middleware to protect server functions:

```ts
import { createServerFn } from "@tanstack/react-start";
import { authMiddleware } from "@/lib/auth-middleware";
import { db } from "@/lib/db";

export const getUserData = createServerFn()
  .middleware([authMiddleware])
  .handler(async ({ context }) => {
    const { session } = context;

    const data = await db.item.findMany({
      where: { userId: session.user.id },
    });

    return data;
  });
```

## Step 9: Protected API Routes

For API routes, use the helper:

```ts
import { createFileRoute } from "@tanstack/react-router";
import { requireAuth } from "@/lib/auth-middleware";
import { db } from "@/lib/db";

export const Route = createFileRoute("/api/items")({
  server: {
    handlers: {
      GET: async ({ request }) => {
        const session = await requireAuth(request);

        const items = await db.item.findMany({
          where: { userId: session.user.id },
        });

        return Response.json(items);
      },
    },
  },
});
```

## Auth Pages (Public)

Create `src/routes/sign-in.tsx`:

```tsx
import { createFileRoute, useNavigate } from "@tanstack/react-router";
import { signIn } from "@/lib/auth-client";
import { useState } from "react";

export const Route = createFileRoute("/sign-in")({
  component: SignInPage,
});

function SignInPage() {
  const navigate = useNavigate();
  const [error, setError] = useState("");

  async function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);

    const result = await signIn.email({
      email: formData.get("email") as string,
      password: formData.get("password") as string,
      callbackURL: "/dashboard",
    });

    if (result.error) {
      setError(result.error.message);
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" type="email" placeholder="Email" required />
      <input name="password" type="password" placeholder="Password" required />
      {error && <p>{error}</p>}
      <button type="submit">Sign In</button>
    </form>
  );
}
```

## Redirect Authenticated Users from Auth Pages

Update sign-in to redirect logged-in users:

```tsx
export const Route = createFileRoute("/sign-in")({
  beforeLoad: async () => {
    const session = await getSessionFn();

    if (session) {
      throw redirect({ to: "/dashboard" });
    }
  },
  component: SignInPage,
});
```

## Environment Variables

```env
BETTER_AUTH_SECRET=<random-32-char-secret>
BETTER_AUTH_URL=http://localhost:3000
DATABASE_URL=postgresql://...
VITE_APP_URL=http://localhost:3000
```

## File Structure

```
src/
  lib/
    auth.ts              # Server auth config (with tanstackStartCookies)
    auth-client.ts       # Client auth methods
    auth-middleware.ts   # Session helpers and middleware
  routes/
    api/
      auth/
        $.ts             # Auth API handler
      items.ts           # Protected API example
    _authenticated.tsx   # Protected layout (server-side check)
    _authenticated/
      dashboard.tsx      # Protected page
    sign-in.tsx          # Public auth page
    sign-up.tsx          # Public auth page
```

## Common Mistakes to Avoid

### Wrong: Client-side only protection

```tsx
// DON'T DO THIS - page loads before redirect
function ProtectedLayout() {
  const { data: session, isPending } = useSession();
  const navigate = useNavigate();

  useEffect(() => {
    if (!isPending && !session) {
      navigate({ to: "/sign-in" });
    }
  }, [isPending, session]);

  if (isPending) return <div>Loading...</div>;
  if (!session) return null;

  return <Outlet />;
}
```

### Right: Server-side protection with beforeLoad

```tsx
// DO THIS - blocks route before any content loads
export const Route = createFileRoute("/_authenticated")({
  beforeLoad: async () => {
    const session = await getSessionFn();
    if (!session) {
      throw redirect({ to: "/sign-in" });
    }
    return { session };
  },
  component: AuthenticatedLayout,
});
```

### Wrong: Missing tanstackStartCookies plugin

```ts
// DON'T DO THIS - cookies won't work correctly
export const auth = betterAuth({
  database: prismaAdapter(db, { provider: "postgresql" }),
  emailAndPassword: { enabled: true },
});
```

### Right: Include the cookie plugin

```ts
// DO THIS
import { tanstackStartCookies } from "better-auth/tanstack-start";

export const auth = betterAuth({
  database: prismaAdapter(db, { provider: "postgresql" }),
  emailAndPassword: { enabled: true },
  plugins: [
    tanstackStartCookies(), // Required!
  ],
});
```

## When to Use Client-Side Session

Client-side `useSession()` is fine for:
- Showing/hiding UI elements (logout button, user avatar)
- Displaying user info in headers
- Conditional rendering that doesn't protect sensitive content

It is NOT sufficient for:
- Protecting routes from unauthorized access
- Guarding sensitive data or pages
- Any security-critical check

## tRPC Authentication

If your project uses tRPC, see [trpc.md](trpc.md) for how to create protected procedures.

Quick summary:
1. Add session to tRPC context via `createContext`
2. Create `protectedProcedure` that checks for session
3. Use `protectedProcedure` instead of `publicProcedure` for authenticated routes

## Verification Checklist

- [ ] `tanstackStartCookies` plugin added to auth config
- [ ] Protected routes use `beforeLoad` with server-side session check
- [ ] Server functions use `authMiddleware`
- [ ] API routes use `requireAuth` helper
- [ ] tRPC routes use `protectedProcedure` (if using tRPC)
- [ ] Auth pages redirect authenticated users
- [ ] No client-side-only protection on sensitive routes
