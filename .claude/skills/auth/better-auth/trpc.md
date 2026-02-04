# Better Auth with tRPC

How to authenticate tRPC routes with Better Auth in TanStack Start applications.

## Overview

tRPC procedures can be protected by:
1. Adding session to the tRPC context
2. Creating a `protectedProcedure` that requires authentication
3. Using protected procedures for any route that needs auth

## Step 1: Add Session to tRPC Context

Update your tRPC initialization to include the session in context.

In `src/lib/trpc/init.ts` (or wherever you initialize tRPC):

```ts
import { initTRPC, TRPCError } from "@trpc/server";
import { auth } from "@/lib/auth";

// Create context with session
export async function createContext(request: Request) {
  const session = await auth.api.getSession({
    headers: request.headers,
  });

  return { session };
}

export type Context = Awaited<ReturnType<typeof createContext>>;

const t = initTRPC.context<Context>().create();

export const router = t.router;
export const publicProcedure = t.procedure;
```

## Step 2: Create Protected Procedure

Add a protected procedure that enforces authentication:

```ts
import { initTRPC, TRPCError } from "@trpc/server";
import { auth } from "@/lib/auth";

export async function createContext(request: Request) {
  const session = await auth.api.getSession({
    headers: request.headers,
  });

  return { session };
}

export type Context = Awaited<ReturnType<typeof createContext>>;

const t = initTRPC.context<Context>().create();

export const router = t.router;
export const publicProcedure = t.procedure;

// Protected procedure - requires authentication
export const protectedProcedure = t.procedure.use(async ({ ctx, next }) => {
  if (!ctx.session) {
    throw new TRPCError({
      code: "UNAUTHORIZED",
      message: "You must be logged in to access this resource",
    });
  }

  return next({
    ctx: {
      session: ctx.session, // session is now guaranteed to exist
    },
  });
});
```

## Step 3: Pass Context to tRPC Handler

Update your tRPC API route to pass the context.

In `src/routes/api/trpc/$.ts`:

```ts
import { createFileRoute } from "@tanstack/react-router";
import { fetchRequestHandler } from "@trpc/server/adapters/fetch";
import { appRouter } from "@/lib/trpc/router";
import { createContext } from "@/lib/trpc/init";

export const Route = createFileRoute("/api/trpc/$")({
  server: {
    handlers: {
      GET: async ({ request }) => {
        return fetchRequestHandler({
          endpoint: "/api/trpc",
          req: request,
          router: appRouter,
          createContext: () => createContext(request),
        });
      },
      POST: async ({ request }) => {
        return fetchRequestHandler({
          endpoint: "/api/trpc",
          req: request,
          router: appRouter,
          createContext: () => createContext(request),
        });
      },
    },
  },
});
```

## Step 4: Use Protected Procedures in Router

In your router, use `protectedProcedure` for any route that needs authentication:

```ts
import { z } from "zod";
import { router, publicProcedure, protectedProcedure } from "./init";
import { db } from "@/lib/db";

export const appRouter = router({
  // Public - anyone can call this
  healthCheck: publicProcedure.query(() => {
    return { status: "ok" };
  }),

  // Protected - requires authentication
  getCurrentUser: protectedProcedure.query(async ({ ctx }) => {
    // ctx.session is guaranteed to exist here
    return {
      id: ctx.session.user.id,
      name: ctx.session.user.name,
      email: ctx.session.user.email,
    };
  }),

  // Protected with database query
  getMyItems: protectedProcedure.query(async ({ ctx }) => {
    const items = await db.item.findMany({
      where: { userId: ctx.session.user.id },
    });
    return items;
  }),

  // Protected mutation
  createItem: protectedProcedure
    .input(z.object({ name: z.string() }))
    .mutation(async ({ ctx, input }) => {
      const item = await db.item.create({
        data: {
          name: input.name,
          userId: ctx.session.user.id,
        },
      });
      return item;
    }),
});

export type AppRouter = typeof appRouter;
```

## Step 5: Call Protected Procedures from Client

On the client, protected procedures work the same as public ones - the auth is handled automatically via cookies:

```tsx
import { trpc } from "@/lib/trpc/client";

function Dashboard() {
  // This will fail with UNAUTHORIZED if not logged in
  const { data: user } = trpc.getCurrentUser.useQuery();
  const { data: items } = trpc.getMyItems.useQuery();

  const createItem = trpc.createItem.useMutation();

  return (
    <div>
      <h1>Welcome, {user?.name}</h1>
      <ul>
        {items?.map((item) => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
      <button onClick={() => createItem.mutate({ name: "New Item" })}>
        Add Item
      </button>
    </div>
  );
}
```

## Complete Example: init.ts

Here's a complete tRPC initialization file with auth:

```ts
import { initTRPC, TRPCError } from "@trpc/server";
import superjson from "superjson";
import { auth } from "@/lib/auth";

// Context creation
export async function createContext(request: Request) {
  const session = await auth.api.getSession({
    headers: request.headers,
  });

  return { session };
}

export type Context = Awaited<ReturnType<typeof createContext>>;

// tRPC initialization
const t = initTRPC.context<Context>().create({
  transformer: superjson,
});

// Base router and procedures
export const router = t.router;
export const publicProcedure = t.procedure;

// Protected procedure middleware
const enforceAuth = t.middleware(async ({ ctx, next }) => {
  if (!ctx.session) {
    throw new TRPCError({
      code: "UNAUTHORIZED",
      message: "You must be logged in",
    });
  }

  return next({
    ctx: {
      session: ctx.session,
    },
  });
});

export const protectedProcedure = t.procedure.use(enforceAuth);

// Optional: Admin-only procedure
const enforceAdmin = t.middleware(async ({ ctx, next }) => {
  if (!ctx.session) {
    throw new TRPCError({ code: "UNAUTHORIZED" });
  }

  // Check admin status in database
  const user = await db.user.findUnique({
    where: { id: ctx.session.user.id },
    select: { role: true },
  });

  if (user?.role !== "admin") {
    throw new TRPCError({ code: "FORBIDDEN", message: "Admin access required" });
  }

  return next({ ctx: { session: ctx.session } });
});

export const adminProcedure = t.procedure.use(enforceAdmin);
```

## Error Handling on Client

Handle unauthorized errors gracefully:

```tsx
import { trpc } from "@/lib/trpc/client";
import { useNavigate } from "@tanstack/react-router";

function ProtectedComponent() {
  const navigate = useNavigate();

  const { data, error } = trpc.getMyItems.useQuery(undefined, {
    retry: false,
    onError: (err) => {
      if (err.data?.code === "UNAUTHORIZED") {
        navigate({ to: "/sign-in" });
      }
    },
  });

  if (error) return <div>Error loading data</div>;

  return <div>{/* render data */}</div>;
}
```

## Security Notes

1. **Always use `protectedProcedure`** for any route that accesses user-specific data
2. **Never trust client-side auth checks** - the tRPC middleware handles this server-side
3. **Always filter by userId** in database queries: `where: { userId: ctx.session.user.id }`
4. **Use `adminProcedure`** for admin-only operations

## Verification Checklist

- [ ] `createContext` function gets session from request headers
- [ ] Context is passed to `fetchRequestHandler`
- [ ] `protectedProcedure` throws UNAUTHORIZED if no session
- [ ] All user-specific routes use `protectedProcedure`
- [ ] Database queries filter by `ctx.session.user.id`
