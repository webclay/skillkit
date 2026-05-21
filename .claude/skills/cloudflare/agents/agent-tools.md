# Agent Tools

Agent tools let a parent agent dispatch work to a child Think or AIChatAgent sub-agent and stream events back. The child owns its own messages, stream, tools, and storage. The parent gets live event replay, abort bridging, and UI drill-in support.

Import from `agents/agent-tools`.

Official docs: https://developers.cloudflare.com/agents/

## When to Use Agent Tools vs Plain Sub-Agents

| Use `agentTool()` when... | Use `subAgent()` directly when... |
|---------------------------|-----------------------------------|
| Parent is an LLM-driven agent and wants to delegate a task to a child agent | Parent code controls the flow, not the LLM |
| You want the LLM to decide when to invoke the child | You are spawning workers programmatically |
| You need live event streaming to the parent's UI | You need simple typed RPC |
| You want built-in abort bridging + run cleanup | The child is not a chat agent |

## Basic Setup

```typescript
import { Think } from "@cloudflare/think";
import { agentTool } from "agents/agent-tools";
import { z } from "zod";

// Child agent - the specialist
export class Researcher extends Think<Env> {
  getModel() {
    return createWorkersAI({ binding: this.env.AI })("@cf/meta/llama-4-scout-17b-16e-instruct");
  }

  getSystemPrompt() {
    return "Research the given topic thoroughly and produce a concise summary.";
  }
}

// Parent agent - the orchestrator
export class Assistant extends Think<Env> {
  getModel() {
    return createWorkersAI({ binding: this.env.AI })("@cf/meta/llama-4-scout-17b-16e-instruct");
  }

  getTools() {
    return {
      research: agentTool(Researcher, {
        description: "Research a topic in depth using a specialist agent.",
        inputSchema: z.object({
          query: z.string().min(3).describe("Research question or topic"),
        }),
        displayName: "Researcher",
      }),
    };
  }
}
```

When the LLM calls the `research` tool, a Researcher sub-agent is spawned, runs a full Think turn with streaming, and the events are replayed to the parent's connected clients in real time.

## runAgentTool() - Programmatic Dispatch

For non-LLM-driven dispatch (parent code decides when to call), use `runAgentTool()` directly:

```typescript
import { runAgentTool } from "agents/agent-tools";

// Inside a parent agent method:
const result = await runAgentTool(this, Researcher, "researcher-1", {
  query: "What are the latest trends in serverless computing?",
});
```

## wrangler.jsonc

Only the parent needs a binding. Child/specialist classes are exported but not in `new_sqlite_classes`:

```jsonc
{
  "durable_objects": {
    "bindings": [
      { "name": "Assistant", "class_name": "Assistant" }
    ]
  },
  "migrations": [
    {
      "tag": "v1",
      "new_sqlite_classes": ["Assistant"]
    }
  ]
}
```

Export all classes from your Worker entry point:

```typescript
export { Assistant } from "./agents/assistant";
export { Researcher } from "./agents/researcher";

export default {
  async fetch(request: Request, env: Env, ctx: ExecutionContext) {
    return (await routeAgentRequest(request, env)) ?? new Response("Not found", { status: 404 });
  },
} satisfies ExportedHandler<Env>;
```

## Live Event Streaming

The parent broadcasts `agent-tool-event` frames to all connected clients as the child Think agent streams its response. On the React side, these events integrate with the standard `useAgentChat` message list automatically.

## Cleanup

Once a tool run is complete and you no longer need the child's retained state, call `clearAgentToolRuns()` from the parent to clean up:

```typescript
// Inside the parent agent:
await this.clearAgentToolRuns();
```

This removes stored run state (not the child agent's full SQLite storage). Call it periodically or after a session ends to avoid unbounded growth.

## Abort Bridging

When the parent's turn is cancelled (via AbortSignal or a cancel message from the client), `agentTool` propagates the abort to the running child turn automatically. No manual wiring required.

## Multi-Level Orchestration

Agent tools can be nested - a parent can have agent tools that themselves are Think agents with their own agent tools:

```typescript
export class Manager extends Think<Env> {
  getTools() {
    return {
      codeReview: agentTool(Reviewer, {
        description: "Run a code review",
        inputSchema: z.object({ pr: z.string() }),
      }),
      research: agentTool(Researcher, {
        description: "Research a topic",
        inputSchema: z.object({ topic: z.string() }),
      }),
    };
  }
}

export class Reviewer extends Think<Env> {
  getSystemPrompt() { return "You are an expert code reviewer."; }
}

export class Researcher extends Think<Env> {
  getSystemPrompt() { return "You are a research specialist."; }
}
```
