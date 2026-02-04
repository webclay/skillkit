# Better Auth with Convex Integration

When using Convex as your database, Better Auth requires the `@convex-dev/better-auth` integration package instead of traditional database adapters.

## Prerequisites

- Convex project set up and running (`npx convex dev`)
- Convex version 1.25.0 or later

## Installation

```bash
bun add @convex-dev/better-auth better-auth@1.4.9
# or
pnpm add @convex-dev/better-auth better-auth@1.4.9
```

**Important:** Pin `better-auth` to version `1.4.9` for compatibility with `@convex-dev/better-auth`.

## Step-by-Step Setup

### 1. Update vite.config.ts

Add SSR configuration to bundle the package:

```ts
export default defineConfig({
  ssr: {
    noExternal: ['@convex-dev/better-auth'],
  },
  // ... other config
})
```

### 2. Create convex/convex.config.ts

Register the Better Auth component:

```ts
import { defineApp } from 'convex/server'
import betterAuth from '@convex-dev/better-auth/convex.config'

const app = defineApp()
app.use(betterAuth)
export default app
```

### 3. Create convex/auth.config.ts

Configure the auth provider:

```ts
import { getAuthConfigProvider } from '@convex-dev/better-auth/auth-config'
import type { AuthConfig } from 'convex/server'

export default {
  providers: [getAuthConfigProvider()],
} satisfies AuthConfig
```

### 4. Update convex/auth.ts

Create the auth instance with Convex adapter:

```ts
import { betterAuth } from 'better-auth/minimal'
import { createClient } from '@convex-dev/better-auth'
import { convex } from '@convex-dev/better-auth/plugins'
import { components } from './_generated/api'
import authConfig from './auth.config'

const siteUrl = process.env.SITE_URL || 'http://localhost:3000'

export const authComponent = createClient(components.betterAuth)

export const createAuth = (ctx: any) => {
  return betterAuth({
    baseURL: siteUrl,
    database: authComponent.adapter(ctx),
    emailAndPassword: {
      enabled: true,
      requireEmailVerification: false,
      minPasswordLength: 8,
    },
    plugins: [convex({ authConfig })],
    user: {
      additionalFields: {
        role: {
          type: 'string',
          defaultValue: 'user',
          required: false,
        },
      },
    },
  })
}

export type Auth = ReturnType<typeof createAuth>
```

### 5. Create convex/http.ts

Mount the auth HTTP routes:

```ts
import { httpRouter } from 'convex/server'
import { authComponent, createAuth } from './auth'

const http = httpRouter()
authComponent.registerRoutes(http, createAuth)

export default http
```

### 6. Update src/lib/auth-client.ts

Use the Convex client plugin:

```ts
import { createAuthClient } from 'better-auth/react'
import { convexClient } from '@convex-dev/better-auth/client/plugins'

export const authClient = createAuthClient({
  plugins: [convexClient()],
})

export const { signIn, signUp, signOut, useSession } = authClient
```

### 7. Create src/lib/auth-server.ts

Set up server utilities for TanStack Start:

```ts
import { convexBetterAuthReactStart } from '@convex-dev/better-auth/react-start'

export const {
  handler,
  getToken,
  fetchAuthQuery,
  fetchAuthMutation,
  fetchAuthAction,
} = convexBetterAuthReactStart({
  convexUrl: process.env.VITE_CONVEX_URL!,
  convexSiteUrl: process.env.VITE_CONVEX_SITE_URL!,
})
```

### 8. Create API Route Handler

For TanStack Start, create `src/routes/api/auth/$.ts`:

```ts
import { createFileRoute } from '@tanstack/react-router'
import { handler } from '@/lib/auth-server'

export const Route = createFileRoute('/api/auth/$')({
  server: {
    handlers: {
      GET: ({ request }) => handler(request),
      POST: ({ request }) => handler(request),
    },
  },
})
```

### 9. Update Convex Provider

Update `src/integrations/convex/provider.tsx`:

```tsx
import { ConvexQueryClient } from '@convex-dev/react-query'
import { ConvexBetterAuthProvider } from '@convex-dev/better-auth/react'
import { authClient } from '@/lib/auth-client'

const CONVEX_URL = import.meta.env.VITE_CONVEX_URL

const convexQueryClient = CONVEX_URL
  ? new ConvexQueryClient(CONVEX_URL, {
      expectAuth: true,
    })
  : null

export default function AppConvexProvider({
  children,
  initialToken,
}: {
  children: React.ReactNode
  initialToken?: string | null
}) {
  if (!convexQueryClient) {
    return <>{children}</>
  }

  return (
    <ConvexBetterAuthProvider
      client={convexQueryClient.convexClient}
      authClient={authClient}
      initialToken={initialToken}
    >
      {children}
    </ConvexBetterAuthProvider>
  )
}
```

### 10. Update Root Component

In `src/routes/__root.tsx`, fetch and pass the token:

```tsx
import { createServerFn } from '@tanstack/react-start'
import { getToken } from '../lib/auth-server'

const getAuth = createServerFn({ method: 'GET' }).handler(async () => {
  return await getToken()
})

export const Route = createRootRouteWithContext<MyRouterContext>()({
  beforeLoad: async () => {
    const token = await getAuth()
    return { token }
  },
  component: RootComponent,
})

function RootComponent() {
  const context = useRouteContext({ from: Route.id })

  return (
    <ConvexProvider initialToken={context.token}>
      <Outlet />
    </ConvexProvider>
  )
}
```

### 11. Set Environment Variables

Add to `.env`:

```env
VITE_CONVEX_URL=https://your-deployment.convex.cloud
VITE_CONVEX_SITE_URL=https://your-deployment.convex.site
SITE_URL=http://localhost:3000
VITE_SITE_URL=http://localhost:3000
```

Set Convex environment variables:

```bash
npx convex env set BETTER_AUTH_SECRET "$(openssl rand -base64 32)"
npx convex env set SITE_URL "http://localhost:3000"
```

### 12. Update Convex Schema

Ensure your `convex/schema.ts` includes the Better Auth tables:

```ts
import { defineSchema, defineTable } from 'convex/server'
import { v } from 'convex/values'

export default defineSchema({
  users: defineTable({
    email: v.string(),
    name: v.optional(v.string()),
    image: v.optional(v.string()),
    emailVerified: v.optional(v.boolean()),
    createdAt: v.optional(v.number()),
    updatedAt: v.optional(v.number()),
    role: v.union(v.literal('admin'), v.literal('user')),
  })
    .index('by_email', ['email'])
    .index('by_role', ['role']),

  sessions: defineTable({
    userId: v.string(),
    token: v.string(),
    expiresAt: v.number(),
    ipAddress: v.optional(v.string()),
    userAgent: v.optional(v.string()),
    createdAt: v.optional(v.number()),
    updatedAt: v.optional(v.number()),
  })
    .index('by_userId', ['userId'])
    .index('by_token', ['token']),

  accounts: defineTable({
    userId: v.string(),
    accountId: v.string(),
    providerId: v.string(),
    accessToken: v.optional(v.string()),
    refreshToken: v.optional(v.string()),
    accessTokenExpiresAt: v.optional(v.number()),
    refreshTokenExpiresAt: v.optional(v.number()),
    scope: v.optional(v.string()),
    idToken: v.optional(v.string()),
    password: v.optional(v.string()),
    createdAt: v.optional(v.number()),
    updatedAt: v.optional(v.number()),
  })
    .index('by_userId', ['userId'])
    .index('by_providerId_accountId', ['providerId', 'accountId']),

  verifications: defineTable({
    identifier: v.string(),
    value: v.string(),
    expiresAt: v.number(),
    createdAt: v.optional(v.number()),
    updatedAt: v.optional(v.number()),
  })
    .index('by_identifier', ['identifier']),
})
```

### 13. Generate Types

Run Convex dev to generate the required types:

```bash
npx convex dev --once
```

## Key Differences from Standard Better Auth

| Standard Better Auth | Convex Integration |
|---------------------|-------------------|
| Uses Prisma/Drizzle adapter | Uses `@convex-dev/better-auth` adapter |
| Database in `lib/auth.ts` | Database adapter in `convex/auth.ts` |
| Auth routes via `auth.handler()` | Auth routes via `authComponent.registerRoutes()` |
| Standard `ConvexProvider` | `ConvexBetterAuthProvider` |
| No HTTP router needed | Requires `convex/http.ts` |
| Secret in `.env` | Secret set via `npx convex env set` |

## Common Issues

### "User not found" on login
The user exists in your custom users table but not in Better Auth's system. With Convex integration, Better Auth manages its own user records. Delete the user from Convex dashboard and register again.

### TypeScript errors with `GenericCtx`
Use `any` type for the ctx parameter in `createAuth`:
```ts
export const createAuth = (ctx: any) => { ... }
```

### "components.betterAuth is undefined"
Run `npx convex dev` to generate types after creating `convex/convex.config.ts`.

### Auth not persisting after page reload
Ensure `initialToken` is passed from root component to the provider.

## Reference

- Official docs: https://labs.convex.dev/better-auth
- Framework guide: https://labs.convex.dev/better-auth/framework-guides/tanstack-start
