# agents/chat-sdk

The `agents/chat-sdk` entrypoint provides a `StateAdapter` for the Cloudflare Chat SDK backed by Agents sub-agents. It stores Chat SDK subscriptions, concurrency locks, pending queues, generic cache entries, callback metadata, thread and channel state, persisted message history, and transcript lists in Durable Object SQLite - sharded through `parent.subAgent()`.

Available from `agents >= 0.13.2`.

Official docs: https://developers.cloudflare.com/agents/

## When to Use

Use `agents/chat-sdk` when you are building a messenger or chat inbox application with the Cloudflare Chat SDK and need durable state for chat infrastructure (threads, channels, message history, subscriptions). It lets a messenger ingress Agent keep Chat SDK state inside child facets instead of requiring a separate top-level DO binding for every state shard.

If you just need a simple AI chatbot, use `@cloudflare/ai-chat` or `@cloudflare/think` directly - no Chat SDK adapter needed.

## Setup

```typescript
import { createChatSdkState, ChatSdkStateAgent } from "agents/chat-sdk";

// Export the state agent class from your Worker entry point
export { ChatSdkStateAgent } from "agents/chat-sdk";

const chat = new Chat({
  adapters,
  state: createChatSdkState(),
});
```

`createChatSdkState()` defaults `parent` from `getCurrentAgent()` when called inside an Agent lifecycle method, so no explicit parent reference is needed in the common case.

## wrangler.jsonc

`ChatSdkStateAgent` is a facet-only class. Export it from your entry point but do NOT add it to `new_sqlite_classes` unless you also use it as a standalone top-level agent:

```jsonc
{
  "durable_objects": {
    "bindings": [
      { "name": "InboxAgent", "class_name": "InboxAgent" }
    ]
  },
  "migrations": [
    {
      "tag": "v1",
      "new_sqlite_classes": ["InboxAgent"]
    }
  ]
}
```

## Custom State Agent

Applications that need custom state behavior can subclass and pass explicitly:

```typescript
import { ChatSdkStateAgent, ChatSdkStateAdapter } from "agents/chat-sdk";

export class MyStateShard extends ChatSdkStateAgent {
  // Override methods for custom behavior
}

const chat = new Chat({
  adapters,
  state: createChatSdkState({
    agent: MyStateShard,
    parent: this,
  }),
});
```

## Exports

| Export | Description |
|---|---|
| `createChatSdkState()` | Convenience factory for Chat SDK state |
| `ChatSdkStateAdapter` | Concrete adapter implementation |
| `ChatSdkStateAgent` | Default sub-agent for durable Chat SDK state |
| `defaultThreadShard()` | Default thread sharding helper |
| `defaultKeyShard()` | Default key sharding helper |
