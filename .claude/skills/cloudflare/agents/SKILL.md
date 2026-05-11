---
name: cloudflare-agents
description: Build stateful AI applications on Cloudflare with the Agents SDK. Covers Agent class, AIChatAgent for streaming chat, state management (setState, SQL), WebSocket connections, tools (server/client/human-in-the-loop), scheduling (delayed, cron, interval), React hooks (useAgent, useAgentChat), wrangler.jsonc configuration, and AI model integration (Workers AI, OpenAI, Anthropic via AI SDK). Trigger words - cloudflare agents, agents sdk, AIChatAgent, stateful agent, agent state, agent tools, agent scheduling, cloudflare ai chat, durable agent, useAgent, useAgentChat, cloudflare durable objects agent, ai agent cloudflare, cloudflare chat, workers agent
---

# Cloudflare Agents SDK

Build stateful AI applications on Cloudflare Workers using the Agents SDK. Each agent runs on a Durable Object with built-in SQLite storage, WebSocket connections, scheduling, and streaming AI chat - all without managing infrastructure.

## Source of Truth

Official documentation: https://developers.cloudflare.com/agents/

Always check the official docs when the skill content conflicts or when you need the latest API details.

## When to Use This Skill

- Building a stateful AI agent or chatbot on Cloudflare
- Adding real-time streaming chat with automatic message persistence
- Creating agents with tool calling (server-side, client-side, or human-in-the-loop approval)
- Scheduling delayed, cron, or interval tasks from an agent
- Using WebSocket connections with Durable Objects for real-time features
- Connecting a React frontend to a Cloudflare agent
- Building multi-agent systems with sub-agents

## When NOT to Use This Skill

- Deploying a framework to Cloudflare Workers without agents (use `cloudflare/tanstack-start`, `cloudflare/astro`, etc.)
- Simple stateless API endpoints (use `cloudflare/workers-core`)
- Using Vercel AI SDK without Cloudflare (use `ai/vercel-ai-sdk`)
- Raw Durable Objects without the Agents SDK convenience layer (use `cloudflare/workers-core` Durable Objects binding)

## Prerequisites

Read [workers-core](../workers-core/SKILL.md) for wrangler CLI, bindings, and deployment fundamentals.

## Setup

### Install Packages

```bash
# Base agents package (required)
npm i agents

# For AI chat agents (streaming chat with message persistence)
npm i @cloudflare/ai-chat

# For AI SDK integration (streamText, tool definitions)
npm i ai workers-ai-provider zod

# For Hono integration (optional)
npm i hono-agents
```

### TypeScript Configuration

Extend the agents TypeScript config:

```json
{
  "extends": "agents/tsconfig"
}
```

### Vite Plugin

For Vite-based projects, add the agents plugin to `vite.config.ts`:

```typescript
import agents from "agents/vite";

export default defineConfig({
  plugins: [agents()]
});
```

### wrangler.jsonc

Every agent needs a Durable Object binding and a SQLite migration:

```jsonc
{
  "name": "my-agent-app",
  "main": "src/server.ts",
  "compatibility_flags": ["nodejs_compat"],
  // Workers AI - no API key needed
  "ai": {
    "binding": "AI"
  },
  "durable_objects": {
    "bindings": [
      {
        "name": "MyAgent",
        "class_name": "MyAgent"
      }
    ]
  },
  "migrations": [
    {
      "tag": "v1",
      "new_sqlite_classes": ["MyAgent"]
    }
  ]
}
```

Key rules:
- The binding `name` becomes `env.MyAgent` in your code
- `class_name` must exactly match the exported class name
- Use `new_sqlite_classes` (NOT `new_classes`) - agents require SQLite storage
- `nodejs_compat` is mandatory

### Multiple Agents

Register additional agents by adding more bindings and including them in the migration:

```jsonc
{
  "durable_objects": {
    "bindings": [
      { "name": "ChatAgent", "class_name": "ChatAgent" },
      { "name": "TaskAgent", "class_name": "TaskAgent" }
    ]
  },
  "migrations": [
    {
      "tag": "v1",
      "new_sqlite_classes": ["ChatAgent", "TaskAgent"]
    }
  ]
}
```

### Routing

Export your agent class and set up routing in your Worker entry point:

**Vanilla Workers:**

```typescript
import { routeAgentRequest } from "agents";

export { ChatAgent } from "./agents/chat";

export default {
  async fetch(request: Request, env: Env, ctx: ExecutionContext) {
    const agentResponse = await routeAgentRequest(request, env);
    if (agentResponse) return agentResponse;

    // Your existing routes below
    return new Response("Not found", { status: 404 });
  }
} satisfies ExportedHandler<Env>;
```

**Hono:**

```typescript
import { agentsMiddleware } from "hono-agents";

const app = new Hono();
app.use("*", agentsMiddleware());
```

**Custom routing prefix:**

```typescript
const agentResponse = await routeAgentRequest(request, env, {
  prefix: "/api/agents"
});
```

Default route pattern: `/agents/{agent-name}/{instance-name}`

### Scaffolding a New Project

```bash
npx create-cloudflare@latest --template cloudflare/agents-starter
cd agents-starter && npm install
npm run dev
```

This gives you a working chat agent with tools, scheduling, and a React UI at `http://localhost:5173`.

## Agent Class

### Basic Agent

```typescript
import { Agent, callable } from "agents";

type CounterState = {
  count: number;
};

export class CounterAgent extends Agent<Env, CounterState> {
  initialState: CounterState = { count: 0 };

  @callable()
  increment() {
    this.setState({ count: this.state.count + 1 });
    return this.state.count;
  }

  @callable()
  reset() {
    this.setState({ count: 0 });
  }
}
```

- `Agent<Env, State>` is the generic base class
- `initialState` sets defaults for new agent instances (existing instances load from SQLite)
- `@callable()` makes methods available as typed RPC over WebSocket

### Lifecycle Hooks

```typescript
export class MyAgent extends Agent<Env, MyState> {
  // Called when the agent starts (first request or wake from hibernation)
  async onStart() {}

  // Called for HTTP requests to this agent
  async onRequest(request: Request): Promise<Response> {
    return new Response("OK");
  }

  // WebSocket lifecycle
  onConnect(connection: Connection, ctx: ConnectionContext) {}
  onMessage(connection: Connection, message: string | ArrayBuffer) {}
  onClose(connection: Connection, code: number, reason: string) {}
  onError(connection: Connection, error: Error) {}

  // State change notification
  onStateChanged(state: MyState, source: Connection | "server") {}

  // Email handling (requires email routing binding)
  async onEmail(email: EmailMessage) {}
}
```

### Core Properties

| Property | Type | Description |
|----------|------|-------------|
| `this.env` | `Env` | Environment bindings (KV, R2, D1, AI, etc.) |
| `this.ctx` | `ExecutionContext` | Execution context |
| `this.state` | `State` | Current agent state (read-only, use setState to modify) |
| `this.sql` | Template literal function | Embedded SQLite queries |
| `this.name` | `string` | Instance name/ID |

## AIChatAgent

The `AIChatAgent` class extends `Agent` with built-in chat message persistence, streaming, and automatic reconnection.

### Streaming Chat

```typescript
import { AIChatAgent } from "@cloudflare/ai-chat";
import { streamText, convertToModelMessages } from "ai";
import { createWorkersAI } from "workers-ai-provider";

export class ChatAgent extends AIChatAgent<Env> {
  async onChatMessage(onFinish: StreamTextOnFinishCallback) {
    const workersai = createWorkersAI({ binding: this.env.AI });

    const result = streamText({
      model: workersai("@cf/meta/llama-4-scout-17b-16e-instruct"),
      system: "You are a helpful assistant.",
      messages: convertToModelMessages(this.messages),
      onFinish,
    });

    return result.toUIMessageStreamResponse();
  }
}
```

Key points:
- `this.messages` contains the full conversation history (auto-persisted in SQLite)
- `onFinish` callback must be passed to `streamText` for message persistence to work
- `convertToModelMessages()` converts stored messages to the format AI SDK expects
- `toUIMessageStreamResponse()` returns a streaming response the React hooks can consume
- If a client disconnects mid-stream, the agent keeps running and the client can resume

### With Tools

```typescript
import { tool } from "ai";
import { z } from "zod";

export class ChatAgent extends AIChatAgent<Env> {
  async onChatMessage(onFinish: StreamTextOnFinishCallback) {
    const workersai = createWorkersAI({ binding: this.env.AI });

    const result = streamText({
      model: workersai("@cf/meta/llama-4-scout-17b-16e-instruct"),
      system: "You are a helpful assistant with tools.",
      messages: convertToModelMessages(this.messages),
      tools: {
        getWeather: tool({
          description: "Get weather for a city",
          parameters: z.object({
            city: z.string().describe("City name"),
          }),
          execute: async ({ city }) => {
            return { temperature: 22, conditions: "sunny", city };
          },
        }),
      },
      onFinish,
    });

    return result.toUIMessageStreamResponse();
  }
}
```

## AI Model Integration

### Workers AI (No API Key)

Free, built-in, no configuration beyond the binding:

```typescript
import { createWorkersAI } from "workers-ai-provider";

const workersai = createWorkersAI({ binding: this.env.AI });
const result = streamText({
  model: workersai("@cf/meta/llama-4-scout-17b-16e-instruct"),
  messages: convertToModelMessages(this.messages),
});
```

wrangler.jsonc binding:
```jsonc
{ "ai": { "binding": "AI" } }
```

### OpenAI

```typescript
import { openai } from "@ai-sdk/openai";

// Install: npm i @ai-sdk/openai
// Set OPENAI_API_KEY in wrangler.jsonc vars or secrets

const result = streamText({
  model: openai("gpt-4o"),
  messages: convertToModelMessages(this.messages),
});
```

### Anthropic

```typescript
import { anthropic } from "@ai-sdk/anthropic";

// Install: npm i @ai-sdk/anthropic
// Set ANTHROPIC_API_KEY in wrangler.jsonc vars or secrets

const result = streamText({
  model: anthropic("claude-sonnet-4-20250514"),
  messages: convertToModelMessages(this.messages),
});
```

### Gemini (via OpenAI-compatible endpoint)

```typescript
import OpenAI from "openai";

const client = new OpenAI({
  apiKey: this.env.GEMINI_API_KEY,
  baseURL: "https://generativelanguage.googleapis.com/v1beta/openai/",
});
```

### AI Gateway

Route requests through AI Gateway for caching, rate limiting, and observability:

```typescript
const response = await this.env.AI.run(
  "@cf/meta/llama-4-scout-17b-16e-instruct",
  { prompt: "Your prompt" },
  {
    gateway: {
      id: "my-gateway-id",
      skipCache: false,
      cacheTtl: 3360,
    },
  },
);
```

## State Management

### setState and state

`setState()` does three things atomically:
1. Persists to SQLite (survives restarts)
2. Broadcasts to all connected WebSocket clients
3. Triggers `onStateChanged()` on the agent

```typescript
// Replace entire state
this.setState({
  players: ["Alice", "Bob"],
  score: 0,
  status: "playing",
});

// Update specific fields (spread existing state)
this.setState({
  ...this.state,
  score: this.state.score + 10,
});
```

**State must be JSON-serializable.** Plain objects, arrays, strings, numbers, booleans work. Functions, class instances, Maps, Sets, and circular references do not. Use ISO strings for dates.

### onStateChanged

React to state changes with source tracking:

```typescript
onStateChanged(state: MyState, source: Connection | "server") {
  if (source === "server") return; // Ignore own updates
  console.log(`Client ${source.id} updated state`);
}
```

### validateStateChange

Synchronous validation before state is persisted. Throw to reject:

```typescript
validateStateChange(nextState: MyState, source: Connection | "server") {
  if (nextState.score < 0) {
    throw new Error("Score cannot be negative");
  }
}
```

### SQL Storage

Each agent instance has its own embedded SQLite database for structured data beyond simple state:

```typescript
// Template literal queries with parameterized values
type User = { id: string; name: string; email: string };
const [user] = this.sql<User>`SELECT * FROM users WHERE id = ${userId}`;

// Create tables
this.sql`CREATE TABLE IF NOT EXISTS logs (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  action TEXT NOT NULL,
  timestamp TEXT DEFAULT (datetime('now'))
)`;

// Insert
this.sql`INSERT INTO logs (action) VALUES (${action})`;
```

### When to Use State vs SQL

| Use State for... | Use SQL for... |
|------------------|----------------|
| UI state, counters, status | Historical data, logs |
| Active sessions, config | Large collections |
| Real-time synced values | Relationships, queries |
| Small, frequently changing data | Structured, queryable records |

## Tools

Three types of tools supported in agents:

### Server-Side Tools (Auto-Execute)

Run on the server automatically when the LLM calls them:

```typescript
getWeather: tool({
  description: "Get current weather for a city",
  parameters: z.object({
    city: z.string().describe("City name"),
  }),
  execute: async ({ city }) => {
    const response = await fetch(`https://api.weather.com/${city}`);
    return response.json();
  },
}),
```

### Client-Side Tools (Browser)

No `execute` function - the browser handles execution. Results feed back to the LLM automatically:

```typescript
// In agent (server)
getUserTimezone: tool({
  description: "Get the user's current timezone",
  parameters: z.object({}),
}),

// In React component (client)
const { messages } = useAgentChat({
  agent,
  onToolCall: async ({ toolCall }) => {
    if (toolCall.toolName === "getUserTimezone") {
      return Intl.DateTimeFormat().resolvedOptions().timeZone;
    }
  },
});
```

### Human-in-the-Loop (Approval Required)

Pauses execution until the user approves or rejects:

```typescript
calculate: tool({
  description: "Perform a calculation",
  parameters: z.object({
    a: z.number(),
    b: z.number(),
    operation: z.enum(["add", "subtract", "multiply", "divide"]),
  }),
  needsApproval: async ({ a, b }) => {
    // Require approval for large numbers
    return a > 1000 || b > 1000;
  },
  execute: async ({ a, b, operation }) => {
    switch (operation) {
      case "add": return a + b;
      case "subtract": return a - b;
      case "multiply": return a * b;
      case "divide": return a / b;
    }
  },
}),
```

The React UI can render approval UI by checking `part.state === "approval-required"` on tool invocation parts.

## Scheduling

Agents can schedule tasks to run later. Tasks persist in SQLite and survive restarts.

### schedule() - One-Time or Cron

```typescript
// Delayed - run after N seconds
await this.schedule(60, "processData", { itemId: "abc" });

// Date-based - run at specific time
await this.schedule(new Date("2026-06-01T08:00:00Z"), "sendReport", { type: "monthly" });

// Cron - recurring pattern (minute hour day month weekday)
await this.schedule("0 8 * * 1-5", "morningDigest", { userId: "123" });
```

Common cron patterns:
- `"* * * * *"` - every minute
- `"0 * * * *"` - every hour
- `"0 0 * * *"` - daily at midnight
- `"0 8 * * 1-5"` - weekdays at 8am
- `"0 0 1 * *"` - first of every month

### scheduleEvery() - Fixed Interval

```typescript
// Run every 300 seconds (5 minutes)
await this.scheduleEvery(300, "checkForUpdates", { source: "api" });
```

If a callback takes longer than the interval, the next execution is skipped (not queued).

### Managing Schedules

```typescript
// List all schedules
const schedules = await this.listSchedules();

// List by type
const crons = await this.listSchedules({ type: "cron" });

// Get by ID
const schedule = await this.getScheduleById(scheduleId);

// Cancel
await this.cancelSchedule(scheduleId);
```

### Idempotency

- **Cron schedules** are idempotent by default - same callback + cron + payload returns the existing schedule
- **Delayed/date-based** schedules need explicit `idempotent: true`:
  ```typescript
  await this.schedule(60, "processData", payload, { idempotent: true });
  ```
- **Intervals** are idempotent by callback name + interval + payload

### Schedule Callbacks

Scheduled callbacks must be methods on the agent class:

```typescript
export class MyAgent extends Agent<Env, MyState> {
  async processData(payload: { itemId: string }) {
    const item = this.sql`SELECT * FROM items WHERE id = ${payload.itemId}`;
    // Process...
    this.broadcast(JSON.stringify({ type: "processed", itemId: payload.itemId }));
  }
}
```

### Limits

- Max tasks per agent: tens of thousands
- Task payload: up to 2 MB
- Cron precision: minute-level
- Interval precision: second-level

## React Frontend Integration

### useAgent - State Sync

```typescript
import { useAgent } from "agents/react";

function GameUI() {
  const agent = useAgent({
    agent: "GameAgent",
    name: "room-123",
    onStateUpdate: (state) => {
      console.log("State updated:", state);
    },
  });

  const addPlayer = (name: string) => {
    agent.setState({
      ...agent.state,
      players: [...agent.state.players, name],
    });
  };

  return <div>Players: {agent.state?.players.join(", ")}</div>;
}
```

### useAgentChat - Chat UI

```typescript
import { useAgent, useAgentChat } from "agents/react";

function ChatUI() {
  const agent = useAgent({ agent: "ChatAgent" });

  const { messages, input, handleInputChange, handleSubmit } = useAgentChat({
    agent,
    onToolCall: async ({ toolCall }) => {
      if (toolCall.toolName === "getUserTimezone") {
        return Intl.DateTimeFormat().resolvedOptions().timeZone;
      }
    },
  });

  return (
    <div>
      {messages.map((msg) => (
        <div key={msg.id}>{msg.content}</div>
      ))}
      <form onSubmit={handleSubmit}>
        <input value={input} onChange={handleInputChange} />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

### useAgent with Callable Methods

```typescript
const agent = useAgent({ agent: "CounterAgent" });

// Call @callable() methods directly
const count = await agent.call("increment");
```

### Vanilla JavaScript (AgentClient)

```typescript
import { AgentClient } from "agents/client";

const client = new AgentClient({
  agent: "GameAgent",
  name: "room-123",
  onStateUpdate: (state) => {
    document.getElementById("score").textContent = state.score;
  },
});

client.setState({ ...client.state, score: 100 });
```

### Server-Side Agent Access

Access an agent instance from your Worker fetch handler:

```typescript
import { getAgentByName } from "agents";

const agent = getAgentByName(env.MyAgent, "instance-id");
const result = await agent.increment();
```

## Limits

| Resource | Limit |
|----------|-------|
| State per agent | 1 GB |
| CPU per event | 30 seconds (refreshes on each HTTP request / WebSocket message) |
| Wall clock duration | Unlimited (waiting for external ops does not count) |
| Concurrent agents per account | Tens of millions |
| Agent definitions per account | ~250,000+ |
| Script size | 10 MB (Workers Paid Plan) |
| Schedule payloads | 2 MB per task |
| Schedules per agent | Tens of thousands |

The 30-second CPU limit resets on every incoming HTTP request, WebSocket message, or scheduled task execution. Wall clock time for waiting on LLM responses, database queries, or API calls is unlimited.

## Common Gotchas

1. **Use `new_sqlite_classes` not `new_classes`** in migrations - agents require SQLite storage. Using `new_classes` creates a Durable Object without SQLite and the agent SDK will fail.

2. **State must be JSON-serializable** - no Date objects (use ISO strings), no Maps/Sets (use plain objects/arrays), no functions or class instances.

3. **`routeAgentRequest()` is required** in your Worker's fetch handler for WebSocket upgrades and agent HTTP requests to work. Without it, agents are unreachable.

4. **Agent names are case-sensitive** - `useAgent({ agent: "ChatAgent" })` must match the exact binding name in wrangler.jsonc.

5. **Workers AI binding is separate** - `ai: { binding: "AI" }` goes at the root of wrangler.jsonc, NOT inside `durable_objects`.

6. **`nodejs_compat` is mandatory** - add to `compatibility_flags` or agents will fail with cryptic import errors.

7. **Each agent instance is a Durable Object** - state is per-instance (per name/ID), not global. Two users connecting to different instance names get separate state.

8. **Schedule callbacks must be agent methods** - you cannot schedule arbitrary functions. The callback parameter is a method name on the agent class (type-safe via `keyof this`).

9. **Don't forget to export agent classes** - the agent class must be exported from your Worker entry point file for Durable Object routing to discover it.

10. **Dev vs Preview** - `wrangler dev` runs your Worker locally. Use `wrangler dev --remote` or the Vite dev server for testing with actual Workers AI models and bindings.

## Reference Files

Read these when working on specific topics:

- [advanced.md](advanced.md) - Sub-agents, workflows, browser tools, MCP server/client, WebSocket details
- [patterns.md](patterns.md) - Architecture patterns (chat agent, tool-using agent, orchestrator-workers, scheduled processing, multi-model routing)
