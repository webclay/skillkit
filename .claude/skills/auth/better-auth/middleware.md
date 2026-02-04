# Better Auth Route Protection

## Security Principle

**Authentication must be validated server-side.** This applies to ALL frameworks.

| Layer | Purpose |
|-------|---------|
| Server-side auth | **Security** - Blocks access before data is served |
| Client-side auth | **UX only** - Shows/hides UI elements |

Never rely on client-side session checks alone - they are a security vulnerability.

## Server-Side Session Check

### In API Routes (All Frameworks)

```ts
import { auth } from "@/lib/auth";

export async function GET(request: Request) {
  const session = await auth.api.getSession({
    headers: request.headers,
  });

  if (!session) {
    return new Response("Unauthorized", { status: 401 });
  }

  // Use session.user.id for queries
  const data = await db.item.findMany({
    where: { userId: session.user.id },
  });

  return Response.json(data);
}
```

### In Server Components (Next.js)

```tsx
import { auth } from "@/lib/auth";
import { headers } from "next/headers";
import { redirect } from "next/navigation";

export default async function DashboardPage() {
  const session = await auth.api.getSession({
    headers: await headers(),
  });

  if (!session) {
    redirect("/sign-in");
  }

  return <div>Welcome, {session.user.name}!</div>;
}
```

### In Server Actions

```ts
"use server";

import { auth } from "@/lib/auth";
import { headers } from "next/headers";

export async function updateProfile(formData: FormData) {
  const session = await auth.api.getSession({
    headers: await headers(),
  });

  if (!session) {
    throw new Error("Unauthorized");
  }

  // Update user
  await db.user.update({
    where: { id: session.user.id },
    data: { name: formData.get("name") as string },
  });
}
```

## Next.js Middleware

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

## TanStack Start Route Protection

**For TanStack Start, see [tanstack-start.md](tanstack-start.md) for complete setup.**

TanStack Start requires server-side protection using `beforeLoad`:

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
      <header>Welcome, {session.user.name}</header>
      <Outlet />
    </div>
  );
}
```

**Do NOT use this pattern (client-side only - insecure):**

```tsx
// WRONG - page loads before redirect happens
function AppLayout() {
  const { data: session, isPending } = useSession();
  const navigate = useNavigate();

  useEffect(() => {
    if (!isPending && !session) {
      navigate({ to: "/sign-in" });
    }
  }, [isPending, session, navigate]);

  if (isPending) return <div>Loading...</div>;
  if (!session) return null;

  return <Outlet />;
}
```

## Client-Side Session (UX Only)

Client-side `useSession()` is appropriate for UI purposes only:

```tsx
import { useSession } from "@/lib/auth-client";

function Header() {
  const { data: session } = useSession();

  return (
    <header>
      {session ? (
        <span>Welcome, {session.user.name}</span>
      ) : (
        <a href="/sign-in">Sign In</a>
      )}
    </header>
  );
}
```

This is fine for showing/hiding UI elements, but **never use this as your only protection for routes or data**.

## Role-Based Access

```ts
export async function requireRole(role: string) {
  const session = await auth.api.getSession({
    headers: await headers(),
  });

  if (!session) {
    throw new Error("Unauthorized");
  }

  const user = await db.user.findUnique({
    where: { id: session.user.id },
    select: { role: true },
  });

  if (user?.role !== role) {
    throw new Error("Forbidden");
  }

  return session;
}

// Usage
export async function adminAction() {
  await requireRole("admin");
  // Admin-only logic
}
```
