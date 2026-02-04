---
name: convex
description: Convex real-time backend platform with database, auth, and file storage. Trigger words - convex, real-time database, realtime, collaborative, live updates, instant sync, convex query, convex mutation
---

# Convex

Complete backend platform with real-time sync, TypeScript-first database, and built-in auth.

## When to Use This Skill

- Building real-time collaborative apps
- Need instant data synchronization across clients
- Want end-to-end TypeScript type safety
- Rapid development without backend complexity
- Apps requiring reactive UI updates

## When NOT to Use

- Need SQL/relational database
- Require self-hosted solution
- Heavy batch processing workloads

## Setup

```bash
npx convex dev
```

This starts dev server and opens dashboard. Follow prompts to create project.

### Environment Variables

```env
VITE_CONVEX_URL=https://your-project.convex.cloud
```

## Schema Definition

```typescript
// convex/schema.ts
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  messages: defineTable({
    text: v.string(),
    userId: v.id("users"),
    channelId: v.id("channels"),
    createdAt: v.number(),
  })
    .index("by_channel", ["channelId", "createdAt"]),

  users: defineTable({
    name: v.string(),
    email: v.string(),
    role: v.union(v.literal("admin"), v.literal("user")),
  })
    .index("by_email", ["email"]),
});
```

## Queries (Read Data)

```typescript
// convex/messages.ts
import { query } from "./_generated/server";
import { v } from "convex/values";

export const list = query({
  args: { channelId: v.id("channels") },
  handler: async (ctx, args) => {
    return await ctx.db
      .query("messages")
      .withIndex("by_channel", (q) => q.eq("channelId", args.channelId))
      .order("desc")
      .take(100);
  },
});
```

## Mutations (Write Data)

```typescript
// convex/messages.ts
import { mutation } from "./_generated/server";
import { v } from "convex/values";

export const create = mutation({
  args: { text: v.string(), channelId: v.id("channels") },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error("Not authenticated");

    return await ctx.db.insert("messages", {
      text: args.text,
      channelId: args.channelId,
      userId: identity.subject,
      createdAt: Date.now(),
    });
  },
});

export const remove = mutation({
  args: { id: v.id("messages") },
  handler: async (ctx, args) => {
    await ctx.db.delete(args.id);
  },
});
```

## Actions (External APIs)

```typescript
// convex/actions.ts
import { action } from "./_generated/server";
import { v } from "convex/values";

export const sendEmail = action({
  args: { to: v.string(), subject: v.string(), body: v.string() },
  handler: async (ctx, args) => {
    const response = await fetch("https://api.resend.com/emails", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: `Bearer ${process.env.RESEND_API_KEY}`,
      },
      body: JSON.stringify({
        to: args.to,
        subject: args.subject,
        html: args.body,
      }),
    });
    return { success: response.ok };
  },
});
```

## React Integration

```tsx
// Provider setup
import { ConvexProvider, ConvexReactClient } from "convex/react";

const convex = new ConvexReactClient(import.meta.env.VITE_CONVEX_URL);

function App() {
  return (
    <ConvexProvider client={convex}>
      <YourApp />
    </ConvexProvider>
  );
}

// Using queries (auto-subscribes to real-time updates)
import { useQuery, useMutation } from "convex/react";
import { api } from "../convex/_generated/api";

function MessageList({ channelId }) {
  const messages = useQuery(api.messages.list, { channelId });
  const createMessage = useMutation(api.messages.create);

  if (messages === undefined) return <div>Loading...</div>;

  return (
    <div>
      {messages.map((msg) => (
        <div key={msg._id}>{msg.text}</div>
      ))}
      <button onClick={() => createMessage({ text: "Hello", channelId })}>
        Send
      </button>
    </div>
  );
}
```

## Scheduled Jobs

```typescript
// convex/crons.ts
import { cronJobs } from "convex/server";
import { internal } from "./_generated/api";

const crons = cronJobs();

crons.daily(
  "send daily digest",
  { hourUTC: 9, minuteUTC: 0 },
  internal.emails.sendDailyDigest
);

crons.interval(
  "cleanup old data",
  { hours: 1 },
  internal.tasks.cleanup
);

export default crons;
```

## File Storage

```typescript
// Upload
const generateUploadUrl = useMutation(api.files.generateUploadUrl);
const saveFile = useMutation(api.files.save);

async function handleUpload(file: File) {
  const uploadUrl = await generateUploadUrl();
  const response = await fetch(uploadUrl, {
    method: "POST",
    body: file,
  });
  const { storageId } = await response.json();
  await saveFile({ storageId, name: file.name });
}
```

## Tips

- Use indexes for queries (`.withIndex()` not `.filter()`)
- Actions for external APIs, mutations for database
- `useQuery` returns `undefined` while loading
- Environment variables: `npx convex env set KEY value`

## How to Verify

### Quick Checks
- `npx convex dev` starts without errors
- Dashboard shows functions at dashboard.convex.dev
- Queries return data, mutations update in real-time

### Common Issues
- "Not authenticated": Check auth provider setup
- Query returns undefined forever: Check function name in api import
- Action fails: Ensure using action (not mutation) for fetch calls
