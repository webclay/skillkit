---
name: cloudflare-agents
description: Build stateful AI applications on Cloudflare with the Agents SDK. Covers Agent class, AIChatAgent, Think (opinionated chat base), state management (setState, SQL), WebSocket connections, tools (server/client/human-in-the-loop), scheduling, React hooks (useAgent, useAgentChat), sub-agents (facets), agentTool orchestration, managed fiber jobs, wrangler.jsonc configuration, and AI model integration. Trigger words - cloudflare agents, agents sdk, AIChatAgent, Think agent, stateful agent, agent state, agent tools, agent scheduling, cloudflare ai chat, durable agent, useAgent, useAgentChat, cloudflare durable objects agent, ai agent cloudflare, cloudflare chat, workers agent, sub-agent, facet agent, agentTool, runAgentTool, getCurrentAgent, managed fiber, chat-sdk, agents/chat-sdk, CloudflareAgents, AIChatAgent, MCP server, MCP client, multi-agent
---

# Cloudflare Agents SDK

Build stateful AI applications on Cloudflare Workers using the Agents SDK. Each agent runs on a Durable Object with built-in SQLite storage, WebSocket connections, scheduling, and streaming AI chat - all without managing infrastructure.

## Source of Truth

Official documentation: https://developers.cloudflare.com/agents/
GitHub: https://github.com/cloudflare/agents

Always check the official docs when the skill content conflicts or when you need the latest API details.

## When to Use This Skill

- Building a stateful AI agent or chatbot on Cloudflare
- Adding real-time streaming chat with automatic message persistence
- Creating agents with tool calling (server-side, client-side, or human-in-the-loop approval)
- Scheduling delayed, cron, or interval tasks from an agent
- Using WebSocket connections with Durable Objects for real-time features
- Connecting a React frontend to a Cloudflare agent
- Building multi-agent systems with sub-agents (facets)
- Running sub-agents as streaming tools from a parent agent

## When NOT to Use This Skill

- Deploying a framework to Cloudflare Workers without agents (use `cloudflare/tanstack-start`, `cloudflare/astro`, etc.)
- Simple stateless API endpoints (use `cloudflare/workers-core`)
- Using Vercel AI SDK without Cloudflare (use `ai/vercel-ai-sdk`)
- Raw Durable Objects without the Agents SDK convenience layer (use `cloudflare/workers-core` Durable Objects binding)
- Voice agents with STT/TTS (use `cloudflare/agents-voice`)

## Prerequisites

Read [workers-core](../workers-core/SKILL.md) for wrangler CLI, bindings, and deployment fundamentals.

## Setup

### Install Packages

```bash
# Core agents package (required)
npm i agents

# For AI chat agents (streaming chat with message persistence)
npm i @cloudflare/ai-chat

# For the Think opinionated chat base (preferred over AIChatAgent for new projects)
npm i @cloudflare/think

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
npx create-cloudflare@latest -- --template cloudflare/agents-starter
cd agents-starter && npm install
npm run dev
```

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

### getCurrentAgent()

Access the current agent instance from anywhere within its execution context (e.g. from a utility function called during a request):

```typescript
import { getCurrentAgent } from "agents";

function doSomething() {
  const agent = getCurrentAgent<MyAgent>();
  if (agent) {
    agent.setState({ ...agent.state, processed: true });
  }
}
```

## AIChatAgent

The `AIChatAgent` class extends `Agent` with built-in chat message persistence, streaming, and automatic reconnection.

> For new projects, consider `@cloudflare/think` instead - it has a more complete agentic loop, workspace tools, stream resumption, and lifecycle hooks. See [think.md](think.md).

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

## AI Model Integration

### Workers AI (No API Key)

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
type User = { id: string; name: string; email: string };
const [user] = this.sql<User>`SELECT * FROM users WHERE id = ${userId}`;

this.sql`CREATE TABLE IF NOT EXISTS logs (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  action TEXT NOT NULL,
  timestamp TEXT DEFAULT (datetime('now'))
)`;

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

No `execute` function - the browser handles execution:

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

```typescript
calculate: tool({
  description: "Perform a calculation",
  parameters: z.object({
    a: z.number(),
    b: z.number(),
    operation: z.enum(["add", "subtract", "multiply", "divide"]),
  }),
  needsApproval: async ({ a, b }) => {
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
await this.scheduleEvery(300, "checkForUpdates", { source: "api" });
```

### Managing Schedules

```typescript
const schedules = await this.listSchedules();
const crons = await this.listSchedules({ type: "cron" });
const schedule = await this.getScheduleById(scheduleId);
await this.cancelSchedule(scheduleId);
```

Cron schedules are idempotent by default. Delayed/date-based schedules need `{ idempotent: true }`.

### Schedule Callbacks

Scheduled callbacks must be methods on the agent class:

```typescript
export class MyAgent extends Agent<Env, MyState> {
  async processData(payload: { itemId: string }) {
    // Process...
    this.broadcast(JSON.stringify({ type: "processed", itemId: payload.itemId }));
  }
}
```

## Sub-Agents (Facets)

Sub-agents are child agents co-located inside a parent Durable Object. Each child gets isolated SQLite storage and runs as a facet of the parent.

See [sub-agents.md](sub-agents.md) for the full API.

```typescript
import { Agent, callable } from "agents";

export class Orchestrator extends Agent<Env, {}> {
  @callable()
  async research(topic: string) {
    const researcher = await this.subAgent(Researcher, "researcher-1");
    return researcher.analyze(topic);
  }
}

export class Researcher extends Agent<Env, {}> {
  async analyze(topic: string) {
    return { topic, result: "..." };
  }
}
```

**wrangler.jsonc** - only the parent needs a binding. Facet-only classes must NOT be in `new_sqlite_classes`:

```jsonc
{
  "durable_objects": {
    "bindings": [{ "name": "Orchestrator", "class_name": "Orchestrator" }]
  },
  "migrations": [{
    "tag": "v1",
    "new_sqlite_classes": ["Orchestrator"]
  }]
}
```

## React Frontend Integration

### useAgent - State Sync

```typescript
import { useAgent } from "agents/react";

function GameUI() {
  const agent = useAgent({
    agent: "GameAgent",
    name: "room-123",
    onStateUpdate: (state) => console.log("State updated:", state),
  });

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

### Connecting to a Sub-Agent from the Client

```typescript
const agent = useAgent({
  agent: "Inbox",
  name: userId,
  sub: [{ agent: "Chat", name: chatId }],
});
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
```

### Server-Side Agent Access

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
| Script size | 10 MB (Workers Paid Plan) |
| Schedule payloads | 2 MB per task |
| Schedules per agent | Tens of thousands |

## Common Gotchas

1. **Use `new_sqlite_classes` not `new_classes`** in migrations - agents require SQLite storage. Using `new_classes` creates a Durable Object without SQLite and the agent SDK will fail.

2. **Facet-only child classes must NOT be in `new_sqlite_classes`** - sub-agent child classes are discovered via `ctx.exports`, not as top-level DO namespaces. Only add to `new_sqlite_classes` if the class is also used as a standalone top-level agent.

3. **State must be JSON-serializable** - no Date objects (use ISO strings), no Maps/Sets (use plain objects/arrays), no functions or class instances.

4. **`routeAgentRequest()` is required** in your Worker's fetch handler for WebSocket upgrades and agent HTTP requests to work. Without it, agents are unreachable.

5. **Agent names are case-sensitive** - `useAgent({ agent: "ChatAgent" })` must match the exact binding name in wrangler.jsonc.

6. **Workers AI binding is separate** - `ai: { binding: "AI" }` goes at the root of wrangler.jsonc, NOT inside `durable_objects`.

7. **`nodejs_compat` is mandatory** - add to `compatibility_flags` or agents will fail with cryptic import errors.

8. **Each agent instance is a Durable Object** - state is per-instance (per name/ID), not global. Two users connecting to different instance names get separate state.

9. **Schedule callbacks must be agent methods** - you cannot schedule arbitrary functions. The callback parameter is a method name on the agent class (type-safe via `keyof this`).

10. **Don't forget to export agent classes** - the agent class must be exported from your Worker entry point file for Durable Object routing to discover it.

11. **Use async schedule APIs inside sub-agents** - `getSchedule()` and `getSchedules()` (synchronous, deprecated) throw inside sub-agents. Use `getScheduleById()` and `listSchedules()` (async) instead.

## Reference Files

Read these when working on specific topics:

- [sub-agents.md](sub-agents.md) - Facet API, spawning/managing child agents, identity, access control
- [agent-tools.md](agent-tools.md) - Running sub-agents as streaming tools with `agentTool()` / `runAgentTool()`
- [think.md](think.md) - `@cloudflare/think`: opinionated chat base with agentic loop, workspace, lifecycle hooks
- [fibers.md](fibers.md) - Managed fiber jobs: durable background jobs with idempotency, cancellation, recovery
- [codemode.md](codemode.md) - `@cloudflare/codemode`: let LLMs write JS code instead of calling tools
- [shell.md](shell.md) - `@cloudflare/shell`: virtual filesystem + git for agent workspaces
- [chat-sdk.md](chat-sdk.md) - `agents/chat-sdk`: Chat SDK state adapter backed by sub-agent facets
- [postgres-sessions.md](postgres-sessions.md) - Experimental Postgres-backed session/context providers via Hyperdrive
- [advanced.md](advanced.md) - Workflows, MCP server/client, browser tools, WebSocket details
- [patterns.md](patterns.md) - Architecture patterns (chat agent, orchestrator-workers, agent tool orchestration, etc.)
