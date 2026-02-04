# Better Auth Setup Reference

## Full Installation

```bash
pnpm add better-auth
```

## Server Configuration

Create `lib/auth.ts`:

```ts
import { betterAuth } from "better-auth";
import { prismaAdapter } from "better-auth/adapters/prisma";
import { db } from "./db";

export const auth = betterAuth({
  // Database (optional in v1.4+ for stateless mode)
  database: prismaAdapter(db, { provider: "postgresql" }),

  // Email & password
  emailAndPassword: {
    enabled: true,
    minPasswordLength: 8,
    maxPasswordLength: 128,
    autoSignIn: true,
    requireEmailVerification: true,
    sendResetPassword: async ({ user, url }) => {
      // Send password reset email
    },
  },

  // Email verification
  emailVerification: {
    sendOnSignUp: true,
    autoSignInAfterVerification: true,
    sendVerificationEmail: async ({ user, url }) => {
      // Send verification email
    },
  },

  // Session config
  session: {
    expiresIn: 60 * 60 * 24 * 7, // 7 days
    updateAge: 60 * 60 * 24, // Extend after 1 day
    cookieCache: {
      enabled: true,
      maxAge: 60 * 5, // 5 minutes
    },
  },

  // CSRF protection
  trustedOrigins: [
    "http://localhost:3000",
    "https://yourdomain.com",
  ],

  // Advanced options
  advanced: {
    generateId: "uuid", // or "nanoid"
    cookieChunking: { enabled: true },
  },
});
```

## Stateless Mode (v1.4+)

No database needed - sessions via JWT only:

```ts
export const auth = betterAuth({
  // No database config!
  emailAndPassword: { enabled: true },
});
```

**Trade-offs:**
- Faster (no DB calls)
- Can't revoke sessions centrally
- Good for serverless/edge

## Framework Integration

### Next.js App Router

`app/api/auth/[...all]/route.ts`:
```ts
import { auth } from "@/lib/auth";
import { toNextJsHandler } from "better-auth/next-js";

export const { POST, GET } = toNextJsHandler(auth);
```

### TanStack Start

`src/routes/api/auth/$.ts`:
```ts
import { createFileRoute } from "@tanstack/react-router";
import { auth } from "@/lib/auth";

export const Route = createFileRoute("/api/auth/$")({
  server: {
    handlers: {
      GET: async ({ request }) => auth.handler(request),
      POST: async ({ request }) => auth.handler(request),
    },
  },
});
```

### Remix

`app/routes/api.auth.$.ts`:
```ts
import { auth } from "~/lib/auth.server";

export async function loader({ request }) {
  return auth.handler(request);
}

export async function action({ request }) {
  return auth.handler(request);
}
```

## Client Setup

Create `lib/auth-client.ts`:

```ts
import { createAuthClient } from "better-auth/react";

export const authClient = createAuthClient({
  baseURL: process.env.NEXT_PUBLIC_APP_URL || "http://localhost:3000",
});

export const {
  signIn,
  signUp,
  signOut,
  useSession,
  forgetPassword,
  resetPassword,
} = authClient;
```

## Environment Variables

```env
# Required
BETTER_AUTH_SECRET=<generate-random-32-char-secret>
BETTER_AUTH_URL=http://localhost:3000
DATABASE_URL=postgresql://...

# For TanStack Start (client-side)
VITE_BETTER_AUTH_URL=http://localhost:3000

# For Next.js (client-side)
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Database Setup

Use Better Auth CLI to generate schema:
```bash
npx better-auth generate
npx better-auth migrate
```

Or use Prisma schema from [database.md](database.md).
