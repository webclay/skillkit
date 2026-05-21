# Postgres Session Providers

Experimental Postgres-backed session, context, and search providers for Think agents. Uses external Postgres via Hyperdrive-compatible `pg` clients instead of the default Durable Object SQLite.

> **Experimental** - may have breaking changes. Available from `agents >= 0.13.0`.

Official docs: https://developers.cloudflare.com/agents/

## When to Use Postgres vs SQLite

| Use SQLite (default) when... | Use Postgres when... |
|---|---|
| Each agent instance has its own isolated history | Multiple agent instances share the same session history |
| Low-to-medium message volume | High-volume or large conversation histories |
| No existing Postgres infrastructure | You already have Hyperdrive + Postgres set up |
| Simple deployment (no extra config) | You need external search or vector storage |
| New project | Migrating from an existing system with Postgres data |

## Setup

Requires a Hyperdrive binding pointing to a Postgres database:

```jsonc
// wrangler.jsonc
{
  "hyperdrive": [
    {
      "binding": "DB",
      "id": "your-hyperdrive-id"
    }
  ]
}
```

Install dependencies:

```bash
npm install @cloudflare/think pg
npm install --save-dev @types/pg
```

## Usage with Think

```typescript
import { Think } from "@cloudflare/think";
import { createPostgresSession } from "@cloudflare/think/session/postgres";
import { createWorkersAI } from "workers-ai-provider";

export class MyAgent extends Think<Env> {
  getModel() {
    return createWorkersAI({ binding: this.env.AI })("@cf/meta/llama-4-scout-17b-16e-instruct");
  }

  configureSession(session: Session) {
    return session.withProvider(
      createPostgresSession({ client: this.env.DB })
    );
  }
}
```

## Async Session API

Session APIs now consistently return promises when using external providers. Think handles this automatically, but if you call session methods directly:

```typescript
// Always await session operations when using Postgres providers
await session.addContext("memory", { description: "Learned facts" });
await session.refreshSystemPrompt();
```

The async API works identically with local SQLite providers, so your code is provider-agnostic.

## Context and Search Providers

Postgres providers also cover context blocks and search:

```typescript
configureSession(session: Session) {
  return session
    .withProvider(createPostgresSession({ client: this.env.DB }))
    .withSearchProvider(createPostgresSearch({ client: this.env.DB }));
}
```

## Notes

- Hyperdrive-compatible means any `pg`-compatible client that works with Cloudflare Hyperdrive connection pooling
- The SQLite session API is the recommended default for most use cases
- Postgres providers are most valuable when multiple Think instances need to share the same conversation history (e.g. multi-device sync or agent handoff)
- Check the official docs for the current provider API surface as this is experimental and evolving
