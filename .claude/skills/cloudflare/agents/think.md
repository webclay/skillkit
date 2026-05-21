# @cloudflare/think

`Think` is an opinionated chat agent base class for Cloudflare Workers. It handles the full chat lifecycle - agentic loop, streaming, persistence, client tools, stream resumption - all backed by Durable Object SQLite.

Works as both a **top-level agent** (WebSocket chat protocol for browser clients) and a **sub-agent** (RPC streaming from a parent agent via `agentTool()`).

> **Experimental** - the API surface is stable but may evolve before graduating.

GitHub: https://github.com/cloudflare/agents/tree/main/packages/think

## AIChatAgent vs Think

| | `AIChatAgent` | `Think` |
|---|---|---|
| Package | `@cloudflare/ai-chat` | `@cloudflare/think` |
| Agentic loop | Manual (you call streamText) | Built-in, auto-continues after tool calls |
| Workspace | No | Built-in (SQLite-backed virtual filesystem) |
| Stream resumption | Basic | Full (chunk persistence + replay) |
| Lifecycle hooks | None | `beforeTurn`, `beforeStep`, `onStepFinish`, etc. |
| Sub-agent streaming | No | Yes (`chat()` RPC method) |
| Agent tools | No | Yes (`agentTool()` integration) |
| Extensions | No | Yes (sandboxed dynamic tool loading) |

**Use Think for new projects.** Use AIChatAgent for simple cases where you want direct control over the `streamText` call.

## Quick Start

```typescript
import { Think } from "@cloudflare/think";
import { createWorkersAI } from "workers-ai-provider";

export class MyAgent extends Think<Env> {
  getModel() {
    return createWorkersAI({ binding: this.env.AI })(
      "@cf/moonshotai/kimi-k2.6"
    );
  }

  getSystemPrompt() {
    return "You are a helpful coding assistant.";
  }
}
```

Connect from the browser with `useAgentChat` from `@cloudflare/ai-chat` - same hooks as AIChatAgent.

## Install

```bash
npm i @cloudflare/think @cloudflare/ai-chat agents ai zod workers-ai-provider
```

Peer dependencies: `agents`, `ai` (Vercel AI SDK v6), `zod` (v3.25+ or v4), `@cloudflare/shell` (for workspace).

## wrangler.jsonc

Same as any agent - Think extends `Agent`:

```jsonc
{
  "name": "my-think-agent",
  "main": "src/server.ts",
  "compatibility_flags": ["nodejs_compat"],
  "ai": { "binding": "AI" },
  "durable_objects": {
    "bindings": [{ "name": "MyAgent", "class_name": "MyAgent" }]
  },
  "migrations": [{ "tag": "v1", "new_sqlite_classes": ["MyAgent"] }]
}
```

## Configuration Methods

| Method / Property | Default | Description |
|---|---|---|
| `getModel()` | throws | Return the `LanguageModel` to use |
| `getSystemPrompt()` | careful assistant | System prompt |
| `getTools()` | `{}` | AI SDK `ToolSet` for the agentic loop |
| `maxSteps` | `10` | Max tool-call rounds per turn |
| `sendReasoning` | `true` | Send reasoning chunks to chat clients |
| `configureSession()` | identity | Add context blocks, compaction, search |
| `chatRecovery` | `true` | Wrap turns in `runFiber` for durable execution |
| `waitForMcpConnections` | `false` | Wait for MCP connections before inference |

## Tools

```typescript
import { tool } from "ai";
import { z } from "zod";

export class MyAgent extends Think<Env> {
  getModel() { ... }

  getTools() {
    return {
      search: tool({
        description: "Search the web",
        parameters: z.object({ query: z.string() }),
        execute: async ({ query }) => {
          const results = await fetch(`https://api.search.com?q=${query}`);
          return results.json();
        },
      }),
    };
  }
}
```

Think automatically continues the agentic loop after tool calls (up to `maxSteps`).

## Built-in Workspace

Every Think agent gets `this.workspace` - a virtual filesystem backed by the DO's SQLite storage. File operation tools (`read`, `write`, `edit`, `list`, `find`, `grep`, `delete`) are automatically available to the model on every turn.

```typescript
export class MyAgent extends Think<Env> {
  getModel() { ... }
  // this.workspace is ready to use - no setup needed
  // workspace tools are auto-merged into every chat turn
}
```

Override to add R2 spillover for large files:

```typescript
import { Workspace } from "@cloudflare/shell";

export class MyAgent extends Think<Env> {
  override workspace = new Workspace({
    sql: this.ctx.storage.sql,
    r2: this.env.R2,
    name: () => this.name,
  });
}
```

## Lifecycle Hooks

```typescript
import type { TurnContext, TurnConfig } from "@cloudflare/think";

export class MyAgent extends Think<Env> {
  getModel() { ... }

  // Before streamText - can override model, tools, maxSteps, etc.
  beforeTurn(ctx: TurnContext): TurnConfig | void {
    if (ctx.continuation) {
      return { model: this.cheapModel };
    }
  }

  // Before each model step
  beforeStep(ctx: PrepareStepContext) {
    if (ctx.stepNumber === 0) {
      return { activeTools: ["search"], toolChoice: { type: "tool", toolName: "search" } };
    }
  }

  // Before a tool's execute runs
  beforeToolCall(ctx: ToolCallContext) {
    // Block, allow, or substitute
    if (ctx.toolName === "delete") {
      return { action: "block", reason: "Delete is not allowed" };
    }
  }

  // After tool execution (success or failure)
  afterToolCall(ctx: ToolCallResultContext) {
    if (ctx.success) {
      console.log(`${ctx.toolName} ok in ${ctx.durationMs}ms`);
    }
  }

  // After each step completes
  onStepFinish(ctx: StepContext) {}

  // Per streaming chunk (high-frequency)
  onChunk(ctx: ChunkContext) {}

  // After turn completes + message persisted
  onChatResponse(result: unknown) {}
}
```

`TurnConfig` fields you can override per-turn: `model`, `system`, `messages`, `tools`, `activeTools`, `toolChoice`, `maxSteps`, `stopWhen`, `sendReasoning`, `maxOutputTokens`, `temperature`, `stopSequences`, and more.

`beforeToolCall` can return:
- `{ action: "allow", input? }` - run execute, optionally with substituted input
- `{ action: "block", reason? }` - skip execute; model sees reason as tool output
- `{ action: "substitute", output }` - skip execute; model sees output as tool result

## Session and Context Blocks

```typescript
export class MyAgent extends Think<Env> {
  getModel() { ... }

  configureSession(session: Session) {
    return session
      .withContext("memory", { description: "Learned facts", maxTokens: 2000 })
      .withCachedPrompt();
  }
}
```

## Dynamic Configuration

Persist a JSON-serializable config blob that survives hibernation and restarts:

```typescript
type MyConfig = { modelTier: "fast" | "capable" };

export class MyAgent extends Think<Env> {
  getModel() {
    const tier = this.getConfig<MyConfig>()?.modelTier ?? "fast";
    return createWorkersAI({ binding: this.env.AI })(MODEL_IDS[tier]);
  }
}

// From outside (via @callable or HTTP handler):
await agent.configure({ modelTier: "capable" });
const config = agent.getConfig<MyConfig>();
```

## Sub-Agent Streaming via RPC

When used as a sub-agent via `this.subAgent()` or `agentTool()`, Think exposes a `chat()` method for direct streaming:

```typescript
const agent = await this.subAgent(MyAgent, "thread-1");
await agent.chat("Summarize the project", {
  onStart: ({ requestId }) => console.log("started:", requestId),
  onEvent: (json) => relay(json),
  onDone: () => console.log("done"),
  onError: (error) => console.error(error),
});

// Cancel a running turn:
await agent.cancelChat(requestId, "user cancelled");
```

## Choosing a Turn API

| Scenario | API |
|---|---|
| User sends a message in the browser | `useAgentChat` (automatic via WebSocket) |
| Server code triggers a turn, waits for response | `saveMessages()` |
| Webhook triggers a turn, needs fast acceptance and retry | `submitMessages()` |
| Parent delegates work to child Think | `agentTool()` or `runAgentTool()` |
| Direct streaming RPC to a specific child | `subAgent(...).chat()` |

## Code Execution Tool

Let the LLM write and run JavaScript in a sandboxed Worker:

```typescript
import { createExecuteTool } from "@cloudflare/think/tools/execute";

getTools() {
  return {
    execute: createExecuteTool({ tools: wsTools, loader: this.env.LOADER }),
  };
}
```

Requires `@cloudflare/codemode` and a `worker_loaders` binding in `wrangler.jsonc`.

## MCP Integration

Think inherits MCP client support from `Agent`. Set `waitForMcpConnections` to ensure MCP servers are connected before the inference loop runs:

```typescript
export class MyAgent extends Think<Env> {
  waitForMcpConnections = true; // or { timeout: 10_000 }
}
```

MCP tools are automatically merged into every turn.
