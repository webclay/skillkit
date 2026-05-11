# Agents SDK - Architecture Patterns

Common patterns for building AI applications with the Cloudflare Agents SDK.

## Pattern 1: Simple Chat Agent

Minimal setup - streaming AI chat with Workers AI (no API key needed).

```typescript
// src/server.ts
import { AIChatAgent } from "@cloudflare/ai-chat";
import { streamText, convertToModelMessages } from "ai";
import { createWorkersAI } from "workers-ai-provider";
import { routeAgentRequest } from "agents";

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

export default {
  async fetch(request: Request, env: Env, ctx: ExecutionContext) {
    return (await routeAgentRequest(request, env)) || new Response("Not found", { status: 404 });
  },
} satisfies ExportedHandler<Env>;
```

```typescript
// src/client.tsx
import { useAgent, useAgentChat } from "agents/react";

function Chat() {
  const agent = useAgent({ agent: "ChatAgent" });
  const { messages, input, handleInputChange, handleSubmit } = useAgentChat({ agent });

  return (
    <div>
      {messages.map((msg) => (
        <div key={msg.id}>
          <strong>{msg.role}:</strong> {msg.content}
        </div>
      ))}
      <form onSubmit={handleSubmit}>
        <input value={input} onChange={handleInputChange} placeholder="Type a message..." />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

```jsonc
// wrangler.jsonc
{
  "name": "chat-agent",
  "main": "src/server.ts",
  "compatibility_flags": ["nodejs_compat"],
  "ai": { "binding": "AI" },
  "durable_objects": {
    "bindings": [{ "name": "ChatAgent", "class_name": "ChatAgent" }]
  },
  "migrations": [{ "tag": "v1", "new_sqlite_classes": ["ChatAgent"] }]
}
```

**Best for:** Quick prototypes, internal tools, simple chatbots.

## Pattern 2: Tool-Using Agent

Agent with all three tool types - server-side API calls, client-side browser access, and approval-gated actions.

```typescript
import { AIChatAgent } from "@cloudflare/ai-chat";
import { streamText, convertToModelMessages, tool } from "ai";
import { createWorkersAI } from "workers-ai-provider";
import { z } from "zod";

export class AssistantAgent extends AIChatAgent<Env> {
  async onChatMessage(onFinish: StreamTextOnFinishCallback) {
    const workersai = createWorkersAI({ binding: this.env.AI });

    const result = streamText({
      model: workersai("@cf/meta/llama-4-scout-17b-16e-instruct"),
      system: "You are an assistant that can look up data, check the user's location, and send emails (with approval).",
      messages: convertToModelMessages(this.messages),
      tools: {
        // Server-side: auto-executes
        lookupUser: tool({
          description: "Look up a user by email address",
          parameters: z.object({
            email: z.string().email().describe("User email"),
          }),
          execute: async ({ email }) => {
            const [user] = this.sql`SELECT * FROM users WHERE email = ${email}`;
            return user || { error: "User not found" };
          },
        }),

        // Client-side: browser handles it
        getLocation: tool({
          description: "Get the user's current geographic location",
          parameters: z.object({}),
        }),

        // Human-in-the-loop: requires approval
        sendEmail: tool({
          description: "Send an email to a user",
          parameters: z.object({
            to: z.string().email(),
            subject: z.string(),
            body: z.string(),
          }),
          needsApproval: async () => true,
          execute: async ({ to, subject, body }) => {
            await this.env.EMAIL.send({ to, subject, body });
            return { sent: true, to };
          },
        }),
      },
      onFinish,
    });

    return result.toUIMessageStreamResponse();
  }
}
```

**Best for:** Assistants that interact with APIs, databases, and external services.

## Pattern 3: Orchestrator-Workers

Parent agent dispatches tasks to specialized sub-agents for parallel processing.

```typescript
import { Agent, callable } from "agents";

// Parent orchestrator
export class Orchestrator extends Agent<Env, { status: string }> {
  initialState = { status: "idle" };

  @callable()
  async analyzeCompany(companyName: string) {
    this.setState({ ...this.state, status: "analyzing" });

    // Spawn specialized sub-agents in parallel
    const [financials, news, social] = await Promise.all([
      this.subAgent(FinancialAnalyst, `fin-${companyName}`).then((a) => a.analyze(companyName)),
      this.subAgent(NewsAnalyst, `news-${companyName}`).then((a) => a.analyze(companyName)),
      this.subAgent(SocialAnalyst, `social-${companyName}`).then((a) => a.analyze(companyName)),
    ]);

    const summary = { financials, news, social };
    this.setState({ ...this.state, status: "complete" });
    return summary;
  }
}

// Specialized worker agent
export class FinancialAnalyst extends Agent<Env, {}> {
  async analyze(company: string) {
    // Each sub-agent has its own SQLite for caching
    const cached = this.sql`SELECT * FROM cache WHERE company = ${company}`;
    if (cached.length > 0) return cached[0].data;

    const data = await fetchFinancialData(company);
    this.sql`INSERT INTO cache (company, data) VALUES (${company}, ${JSON.stringify(data)})`;
    return data;
  }
}

export class NewsAnalyst extends Agent<Env, {}> {
  async analyze(company: string) {
    // Separate SQLite, separate state
    return await fetchNewsData(company);
  }
}

export class SocialAnalyst extends Agent<Env, {}> {
  async analyze(company: string) {
    return await fetchSocialData(company);
  }
}
```

```jsonc
// wrangler.jsonc - only the parent needs a binding
{
  "durable_objects": {
    "bindings": [{ "name": "Orchestrator", "class_name": "Orchestrator" }]
  },
  "migrations": [{
    "tag": "v1",
    "new_sqlite_classes": ["Orchestrator", "FinancialAnalyst", "NewsAnalyst", "SocialAnalyst"]
  }]
}
```

**Best for:** Research agents, data aggregation, multi-source analysis, complex workflows.

## Pattern 4: Scheduled Processing Agent

Agent that processes data on a schedule with state tracking.

```typescript
import { Agent, callable } from "agents";

type ProcessorState = {
  lastRun: string | null;
  itemsProcessed: number;
  status: "idle" | "running" | "error";
};

export class DataProcessor extends Agent<Env, ProcessorState> {
  initialState: ProcessorState = {
    lastRun: null,
    itemsProcessed: 0,
    status: "idle",
  };

  async onStart() {
    // Set up recurring schedule on first start (idempotent)
    await this.scheduleEvery(3600, "processNewItems", {});
    await this.schedule("0 0 * * 0", "generateWeeklyReport", {});
  }

  async processNewItems() {
    this.setState({ ...this.state, status: "running" });

    try {
      const items = await this.env.QUEUE.pull();
      for (const item of items) {
        await this.processItem(item);
      }

      this.setState({
        ...this.state,
        status: "idle",
        lastRun: new Date().toISOString(),
        itemsProcessed: this.state.itemsProcessed + items.length,
      });
    } catch (error) {
      this.setState({ ...this.state, status: "error" });
    }
  }

  async generateWeeklyReport() {
    const stats = this.sql`SELECT COUNT(*) as total FROM processed_items WHERE created_at > datetime('now', '-7 days')`;
    this.broadcast(JSON.stringify({ type: "report", stats }));
  }

  @callable()
  async getStatus() {
    return this.state;
  }
}
```

**Best for:** Data pipelines, periodic syncs, report generation, queue processing.

## Pattern 5: Multi-Model Agent

Route between cheap/fast and expensive/powerful models based on task complexity.

```typescript
import { AIChatAgent } from "@cloudflare/ai-chat";
import { streamText, convertToModelMessages, generateText } from "ai";
import { createWorkersAI } from "workers-ai-provider";
import { openai } from "@ai-sdk/openai";

export class SmartRouter extends AIChatAgent<Env> {
  async onChatMessage(onFinish: StreamTextOnFinishCallback) {
    const workersai = createWorkersAI({ binding: this.env.AI });

    // Step 1: Classify complexity with a fast model (free)
    const classification = await generateText({
      model: workersai("@cf/meta/llama-4-scout-17b-16e-instruct"),
      prompt: `Classify this request as "simple" or "complex": "${this.messages[this.messages.length - 1].content}"
        Simple: factual questions, greetings, basic tasks.
        Complex: multi-step reasoning, code generation, analysis, creative writing.
        Reply with just one word.`,
    });

    const isComplex = classification.text.toLowerCase().includes("complex");

    // Step 2: Route to appropriate model
    const model = isComplex
      ? openai("gpt-4o") // Powerful but costs money
      : workersai("@cf/meta/llama-4-scout-17b-16e-instruct"); // Fast and free

    const result = streamText({
      model,
      system: "You are a helpful assistant.",
      messages: convertToModelMessages(this.messages),
      onFinish,
    });

    return result.toUIMessageStreamResponse();
  }
}
```

**Best for:** Cost optimization, production agents with variable complexity, high-volume applications.

## Choosing the Right Pattern

| Scenario | Pattern | Why |
|----------|---------|-----|
| Simple chatbot or Q&A | Simple Chat | Minimal code, free Workers AI |
| Agent that calls APIs or databases | Tool-Using | Server-side tools with type-safe schemas |
| Multi-source research or analysis | Orchestrator-Workers | Parallel sub-agents with isolated storage |
| Periodic data processing | Scheduled Processing | Built-in cron/interval with state tracking |
| High-volume with cost concerns | Multi-Model | Route cheap queries to free models |
| Long-running multi-step tasks | Orchestrator + Workflows | Durable execution with retry (see advanced.md) |
| Real-time collaborative features | Basic Agent + WebSocket | State sync across clients via broadcast |

Patterns can be combined. For example, an orchestrator-workers agent can also use scheduling for periodic research updates, and each worker can use tools for API access.
