---
name: unkey
description: Unkey API key management and rate limiting. Trigger words - unkey, api key, rate limit, rate limiting, api authentication, usage tracking, api quota
---

# Unkey

API key management, rate limiting, and usage analytics for public APIs.

## When to Use This Skill

- User building public APIs for external developers
- User needs to issue API keys to customers
- User wants rate limiting without infrastructure
- User needs usage tracking for billing

## When NOT to Use

- Internal-only APIs (use auth middleware)
- User authentication (use Better Auth)
- Simple throttling (use framework middleware)

## Setup

```bash
pnpm add @unkey/api @unkey/ratelimit
```

### Environment Variables

```env
UNKEY_ROOT_KEY=unkey_...   # From Unkey Dashboard
UNKEY_API_ID=api_...       # Your API identifier
```

## Create API Keys

```typescript
import { Unkey } from "@unkey/api";

const unkey = new Unkey({ rootKey: process.env.UNKEY_ROOT_KEY });

const { result, error } = await unkey.keys.create({
  apiId: process.env.UNKEY_API_ID!,
  prefix: "myapp",              // Key prefix
  name: "Customer ABC",         // Human-readable
  ownerId: "user_123",          // Link to your user
  meta: { plan: "pro" },        // Custom metadata
  expires: Date.now() + 30 * 24 * 60 * 60 * 1000,  // 30 days
  ratelimit: {
    type: "fast",
    limit: 100,
    refillRate: 10,
    refillInterval: 1000
  }
});

// Return key to customer (only time you see it!)
console.log("API Key:", result.key);
```

## Verify API Keys

```typescript
import { verifyKey } from "@unkey/api";

export async function verifyApiKey(key: string) {
  const { result, error } = await verifyKey(key);

  if (error) {
    return { valid: false, error: "Service unavailable" };
  }

  if (!result.valid) {
    // NOT_FOUND | FORBIDDEN | RATE_LIMITED | EXPIRED
    return { valid: false, code: result.code };
  }

  return {
    valid: true,
    ownerId: result.ownerId,
    meta: result.meta
  };
}
```

## API Route Middleware

**lib/unkey.ts:**
```typescript
import { verifyKey } from "@unkey/api";

export async function withApiKey(
  request: Request,
  handler: (ownerId: string, meta: any) => Promise<Response>
) {
  const authHeader = request.headers.get("authorization");

  if (!authHeader?.startsWith("Bearer ")) {
    return Response.json({ error: "Missing API key" }, { status: 401 });
  }

  const key = authHeader.slice(7);
  const { result, error } = await verifyKey(key);

  if (error || !result.valid) {
    const status = result?.code === "RATE_LIMITED" ? 429 : 401;
    return Response.json({ error: result?.code || "Invalid key" }, { status });
  }

  return handler(result.ownerId!, result.meta);
}
```

**Usage:**
```typescript
// app/api/data/route.ts
export async function GET(request: Request) {
  return withApiKey(request, async (ownerId, meta) => {
    const data = await fetchDataForUser(ownerId);
    return Response.json({ data });
  });
}
```

## Standalone Rate Limiting

```typescript
import { Ratelimit } from "@unkey/ratelimit";

const ratelimit = new Ratelimit({
  rootKey: process.env.UNKEY_ROOT_KEY!,
  namespace: "api",
  limit: 10,
  duration: "60s"
});

// Rate limit by any identifier
const { success, remaining } = await ratelimit.limit("user_123");

if (!success) {
  return new Response("Rate limit exceeded", { status: 429 });
}
```

## Key Management

```typescript
// Update key
await unkey.keys.update({
  keyId: "key_123",
  meta: { plan: "enterprise" }
});

// Revoke key
await unkey.keys.delete({ keyId: "key_123" });

// List keys
const { result } = await unkey.keys.list({
  apiId: process.env.UNKEY_API_ID!,
  ownerId: "user_123"
});
```

## Tiered Rate Limits

```typescript
const planLimits = {
  free: { limit: 100, refillRate: 10 },
  pro: { limit: 1000, refillRate: 100 },
  enterprise: { limit: 10000, refillRate: 1000 }
};

async function createKeyForPlan(userId: string, plan: string) {
  const limits = planLimits[plan];

  return await unkey.keys.create({
    apiId: process.env.UNKEY_API_ID!,
    ownerId: userId,
    meta: { plan },
    ratelimit: {
      type: "fast",
      ...limits,
      refillInterval: 1000
    }
  });
}
```

## Security

- Never expose root keys client-side
- Use key prefixes for identification
- Set expiration on keys for rotation
- Store keyId (not full key) in your database

## How to Verify

### Quick Checks
- Create key returns `result.key`
- Verify key returns `result.valid: true`
- Rate limiting triggers after limit reached

### Common Issues
- "Key not found": Check key hasn't been revoked
- Rate limit always triggers: Verify refill settings
- Slow verification: Enable async mode
