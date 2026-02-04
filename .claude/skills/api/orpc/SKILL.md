---
name: orpc
description: oRPC type-safe API framework with OpenAPI generation. Trigger words - orpc, openapi, api documentation, swagger, type-safe api, contract-first, api spec
---

# oRPC

Type-safe RPC framework with native OpenAPI generation. Alternative to tRPC with public API support.

## When to Use This Skill

- User needs type-safe APIs with OpenAPI docs
- User mentions oRPC or contract-first development
- User building public APIs for external consumers
- User wants auto-generated API documentation

## When NOT to Use

- Internal-only APIs (tRPC is simpler)
- Non-TypeScript clients
- Simple CRUD (Server Actions suffice)

## Setup

```bash
pnpm add @orpc/server @orpc/client @orpc/next zod
```

## Project Structure

```
src/
├── server/
│   ├── context.ts      # Shared context
│   ├── procedures/     # API procedures
│   │   ├── user.ts
│   │   └── post.ts
│   ├── middleware/     # Auth, logging
│   ├── router.ts       # Main router
│   └── client.ts       # Type-safe client
└── app/api/rpc/[...orpc]/route.ts
```

## Define Procedures

**server/procedures/user.ts:**
```typescript
import { os, ORPCError } from '@orpc/server';
import { z } from 'zod';

export const getUser = os
  .input(z.object({ id: z.number() }))
  .handler(async ({ input }) => {
    const user = await db.user.findUnique({
      where: { id: input.id }
    });

    if (!user) {
      throw new ORPCError({
        code: 'NOT_FOUND',
        message: 'User not found'
      });
    }

    return user;
  });

export const createUser = os
  .input(z.object({
    email: z.string().email(),
    name: z.string().min(2)
  }))
  .output(z.object({
    id: z.number(),
    email: z.string(),
    name: z.string()
  }))
  .handler(async ({ input }) => {
    return await db.user.create({ data: input });
  });

export const userRouter = {
  get: getUser,
  create: createUser
};
```

## Auth Middleware

**server/middleware/auth.ts:**
```typescript
import { ORPCError } from '@orpc/server';
import { base } from '../context';

export const authedProcedure = base.use(async ({ context, next }) => {
  const token = context.headers.get('authorization')?.split(' ')[1];

  if (!token) {
    throw new ORPCError({
      code: 'UNAUTHORIZED',
      message: 'Missing token'
    });
  }

  const user = await verifyJWT(token);

  return next({
    context: { ...context, user }
  });
});
```

## Create Router

**server/router.ts:**
```typescript
import { userRouter } from './procedures/user';
import { postRouter } from './procedures/post';

export const appRouter = {
  user: userRouter,
  post: postRouter
};

export type AppRouter = typeof appRouter;
```

## Next.js API Route

**app/api/rpc/[...orpc]/route.ts:**
```typescript
import { RPCHandler } from '@orpc/next/app';
import { appRouter } from '@/server/router';
import { db } from '@/lib/db';

const handler = RPCHandler({
  router: appRouter,
  context: async (request) => ({
    headers: request.headers,
    db
  })
});

export const POST = handler;
export const GET = handler;
```

## Create Client

**server/client.ts:**
```typescript
import { createORPCClient, RPCLink } from '@orpc/client';
import type { AppRouter } from './router';

const link = new RPCLink<AppRouter>({
  url: `${process.env.NEXT_PUBLIC_APP_URL}/api/rpc`,
  headers: async () => {
    const token = await getToken();
    return { authorization: token ? `Bearer ${token}` : '' };
  }
});

export const orpc = createORPCClient(link);
```

## Usage

**Server Component:**
```typescript
const users = await orpc.user.list();
```

**Client Component:**
```typescript
'use client';

const user = await orpc.user.create({
  email: 'test@example.com',
  name: 'Test'
});
```

## Generate OpenAPI

```typescript
// scripts/generate-openapi.ts
import { generateOpenAPI } from '@orpc/openapi';
import { appRouter } from '@/server/router';

const spec = generateOpenAPI({
  router: appRouter,
  info: { title: 'My API', version: '1.0.0' }
});

fs.writeFileSync('openapi.json', JSON.stringify(spec, null, 2));
```

## Error Codes

- `UNAUTHORIZED` - Not authenticated
- `FORBIDDEN` - Not allowed
- `NOT_FOUND` - Resource missing
- `BAD_REQUEST` - Invalid input
- `CONFLICT` - Duplicate/conflict

## Tips

- Always use Zod for input validation
- Create reusable middleware (auth, logging)
- Export `AppRouter` type for client inference
- Generate OpenAPI docs for external consumers

## How to Verify

### Quick Checks
- `curl -X POST /api/rpc/user.get -d '{"id":1}'` returns user
- TypeScript shows autocomplete on client
- OpenAPI generation succeeds

### Common Issues
- Type errors: Ensure `AppRouter` type is exported
- 404: Check API route path matches
- Context undefined: Verify context function returns properly
