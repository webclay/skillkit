# Advanced Agents SDK Features

Reference: https://developers.cloudflare.com/agents/

## Sub-Agents

Sub-agents are child agents that operate as co-located Durable Objects with isolated SQLite storage. A parent agent coordinates multiple children through typed RPC.

### Spawning Sub-Agents

```typescript
import { Agent, callable } from "agents";

export class Orchestrator extends Agent<Env, OrchestratorState> {
  @callable()
  async research(topic: string) {
    // Spawn or retrieve a named child agent
    const researcher = await this.subAgent(Researcher, "research-1");
    const findings = await researcher.search(topic);
    return findings;
  }
}

export class Researcher extends Agent<Env, ResearcherState> {
  async search(topic: string) {
    // Each child has its own SQLite, state, and scheduling
    this.sql`INSERT INTO searches (topic) VALUES (${topic})`;
    return { topic, results: ["result1", "result2"] };
  }
}
```

Requirements:
- Child class must extend `Agent` and be exported from the Worker entry point
- Export name must match class name exactly
- Only the parent needs a Durable Object binding and migration in wrangler.jsonc
- Child classes are discovered automatically via `ctx.exports`

### Managing Sub-Agents

```typescript
// Check if a sub-agent exists
const exists = this.hasSubAgent(Researcher, "research-1");

// List all sub-agents (optionally filter by class)
const agents = this.listSubAgents(); // all
const researchers = this.listSubAgents(Researcher); // filtered

// Force stop (storage preserved, restarts on next call)
await this.abortSubAgent(Researcher, "research-1");

// Permanently delete (wipes storage)
await this.deleteSubAgent(Researcher, "research-1");
```

### Parallel Execution

```typescript
@callable()
async parallelResearch(topics: string[]) {
  const results = await Promise.all(
    topics.map(async (topic, i) => {
      const agent = await this.subAgent(Researcher, `research-${i}`);
      return agent.search(topic);
    })
  );
  return results;
}
```

### Access Control

Override `onBeforeSubAgent()` on the parent to gate requests:

```typescript
async onBeforeSubAgent(_request: Request, { className, name }: SubAgentInfo) {
  if (!this.hasSubAgent(className, name)) {
    return new Response("Not found", { status: 404 });
  }
  // Return void to allow, or Response to short-circuit
}
```

### Client Routing to Sub-Agents

Connect directly to a sub-agent from the frontend:

```typescript
import { useAgent } from "agents/react";

const chat = useAgent({
  agent: "Inbox",
  name: userId,
  sub: [{ agent: "Chat", name: chatId }],
});
```

### Identity

```typescript
// In a sub-agent:
this.name        // Facet name (e.g., "research-1")
this.parentPath  // Array of ancestors (root-first)
this.selfPath    // Full path including self

// Typed RPC to parent
const parent = this.parentAgent(Orchestrator);
await parent.reportComplete(this.name);
```

Storage is fully isolated - parent SQL and child SQL are completely separate databases.

## Workflows (AgentWorkflow)

Workflows handle durable, multi-step background processing with automatic retry. Use when tasks exceed 30 seconds, need retry logic, or require human approval gates.

### Defining a Workflow

```typescript
import { AgentWorkflow, AgentWorkflowEvent, AgentWorkflowStep } from "agents";

type TaskParams = { data: string; userId: string };

export class ProcessingWorkflow extends AgentWorkflow<MyAgent, TaskParams> {
  async run(event: AgentWorkflowEvent<TaskParams>, step: AgentWorkflowStep) {
    const { data, userId } = event.payload;

    // Step 1: Durable step - won't repeat on retry
    const processed = await step.do("process-data", async () => {
      return processData(data);
    });

    // Step 2: With retry config
    const result = await step.do(
      "call-external-api",
      {
        retries: { limit: 5, delay: "10 seconds", backoff: "exponential" },
        timeout: "5 minutes",
      },
      async () => {
        return await callApi(processed);
      }
    );

    // Update agent state (broadcasts to connected clients)
    await step.updateAgentState({ status: "complete", result });

    // Mark complete
    await step.reportComplete(result);
  }
}
```

### wrangler.jsonc Configuration

```jsonc
{
  "workflows": [
    {
      "name": "processing-workflow",
      "binding": "PROCESSING_WORKFLOW",
      "class_name": "ProcessingWorkflow"
    }
  ]
}
```

### Starting Workflows from Agents

```typescript
export class MyAgent extends Agent<Env, MyState> {
  @callable()
  async startProcessing(data: string) {
    const instanceId = await this.runWorkflow("PROCESSING_WORKFLOW", {
      data,
      userId: "user-456",
    });
    return instanceId;
  }
}
```

### State Updates from Workflows

```typescript
// Replace entire agent state
await step.updateAgentState({ status: "processing", progress: 0 });

// Merge partial updates
await step.mergeAgentState({ progress: 50 });

// Reset to initialState
await step.resetAgentState();
```

Both `updateAgentState` and `mergeAgentState` automatically broadcast changes to connected WebSocket clients.

### Workflow Lifecycle Callbacks

Override these on your agent to react to workflow events:

```typescript
export class MyAgent extends Agent<Env, MyState> {
  async onWorkflowProgress(workflowName: string, instanceId: string, progress: unknown) {
    this.broadcast(JSON.stringify({ type: "progress", progress }));
  }

  async onWorkflowComplete(workflowName: string, instanceId: string, result: unknown) {
    this.setState({ ...this.state, status: "done" });
  }

  async onWorkflowError(workflowName: string, instanceId: string, error: unknown) {
    this.setState({ ...this.state, status: "error" });
  }
}
```

### Human Approval in Workflows

```typescript
// In workflow - wait for approval
const approval = await this.waitForApproval<{ approvedBy: string }>(step, {
  timeout: "7 days",
});

// In agent - approve or reject
await this.approveWorkflow(instanceId, { metadata: { approvedBy: userId } });
await this.rejectWorkflow(instanceId, { reason: "Request denied" });
```

### Managing Workflows

```typescript
// Query workflows
const { workflows, total } = this.getWorkflows({
  status: "running",
  metadata: { userId: "user-456" },
});

// Control execution
await this.terminateWorkflow(instanceId);
await this.pauseWorkflow(instanceId);
await this.resumeWorkflow(instanceId);
await this.restartWorkflow(instanceId);
```

### Workflow Limits

- Max 10,000 steps per workflow (configurable to 25,000)
- 10 MB state size per workflow
- 30-minute execution time per step

## MCP Integration

### Consuming External MCP Tools

Connect to external MCP servers and use their tools in your agent:

```typescript
export class ChatAgent extends AIChatAgent<Env> {
  async onChatMessage(onFinish: StreamTextOnFinishCallback) {
    // Connect to an MCP server
    await this.mcp.connect("https://my-mcp-server.example/sse");

    const result = streamText({
      model: workersai("@cf/meta/llama-4-scout-17b-16e-instruct"),
      messages: convertToModelMessages(this.messages),
      tools: {
        ...myLocalTools,
        ...this.mcp.getAITools(), // Merge MCP tools
      },
      onFinish,
    });

    return result.toUIMessageStreamResponse();
  }
}
```

### Exposing Agent as MCP Server

Your agent's tools can be exposed as an MCP server for other AI systems to consume. Use the `McpAgent` class or `createMcpHandler` for stateless servers:

```typescript
import { McpAgent } from "agents/mcp";
import { z } from "zod";

export class MyMcpServer extends McpAgent<Env> {
  async init() {
    this.server.tool(
      "get_data",
      "Fetch data from the system",
      { query: z.string().describe("Search query") },
      async ({ query }) => {
        const results = await this.env.DB.prepare(
          "SELECT * FROM data WHERE content LIKE ?"
        ).bind(`%${query}%`).all();
        return {
          content: [{ type: "text", text: JSON.stringify(results) }],
        };
      }
    );
  }
}
```

MCP servers support OAuth authorization via Cloudflare Access or third-party providers (GitHub, Google, Auth0, etc.).

## Browser Tools

Agents can browse the web using Chrome DevTools Protocol (CDP) via Cloudflare's Browser Rendering API.

### Setup

```jsonc
// wrangler.jsonc
{
  "browser": {
    "binding": "BROWSER"
  }
}
```

```typescript
import { createBrowserTools } from "@cloudflare/codemode";

const browserTools = createBrowserTools({
  browser: env.BROWSER,
  loader: env.LOADER,
});
```

### Capabilities

- Inspect DOM structure and computed styles
- Capture screenshots or PDFs
- Scrape rendered content (SPA-friendly)
- Debug frontend issues (network, console errors)
- Performance profiling

### Puppeteer Alternative

For programmatic control without LLM code generation:

```bash
npm i @cloudflare/puppeteer
```

```typescript
import puppeteer from "@cloudflare/puppeteer";

const browser = await puppeteer.launch(this.env.BROWSER);
const page = await browser.newPage();
await page.goto("https://example.com");
const content = await page.content();
await browser.close();
```

### Limitations

- Single session per execution call
- No persistent/authenticated sessions
- JavaScript-only sandbox
- ~6,000 token response truncation

## WebSocket Details

### Connection Lifecycle

```typescript
export class RealtimeAgent extends Agent<Env, MyState> {
  onConnect(connection: Connection, ctx: ConnectionContext) {
    // New WebSocket connection established
    // Send initial state or welcome message
    connection.send(JSON.stringify({ type: "welcome", state: this.state }));
  }

  onMessage(connection: Connection, message: string | ArrayBuffer) {
    const data = JSON.parse(message as string);

    switch (data.type) {
      case "action":
        this.handleAction(connection, data);
        break;
    }
  }

  onClose(connection: Connection, code: number, reason: string) {
    // Client disconnected - cleanup if needed
  }

  onError(connection: Connection, error: Error) {
    console.error("WebSocket error:", error);
  }
}
```

### Broadcasting

```typescript
// Send to ALL connected clients
this.broadcast(JSON.stringify({ type: "update", data: newData }));

// Send to a specific connection
connection.send(JSON.stringify({ type: "private", data: result }));
```

### Streaming AI Responses over WebSocket

This is the pattern that solves the runtime limit concern for long-running AI workloads:

```typescript
async onMessage(connection: Connection, message: string | ArrayBuffer) {
  const msg = JSON.parse(message as string);
  const workersai = createWorkersAI({ binding: this.env.AI });

  const result = streamText({
    model: workersai("@cf/meta/llama-4-scout-17b-16e-instruct"),
    prompt: msg.prompt,
  });

  for await (const chunk of result.textStream) {
    if (chunk) {
      connection.send(JSON.stringify({ type: "chunk", content: chunk }));
    }
  }
  connection.send(JSON.stringify({ type: "done" }));
}
```

Each incoming WebSocket message resets the 30-second CPU timer, and wall clock time waiting for the LLM response is unlimited. The WebSocket connection persists through the Durable Object, so even long-running reasoning models work without hitting timeouts.

### Hibernation

Agents automatically hibernate when idle (no connected clients, no pending schedules). When a new connection or request arrives, the agent wakes up, loads persisted state from SQLite, and continues. This is cost-efficient - you only pay for active compute.

### shouldSendProtocolMessages

Override to control which connections receive state broadcasts:

```typescript
shouldSendProtocolMessages(connection: Connection): boolean {
  // Return false to exclude this connection from automatic state sync
  return connection.metadata?.role !== "observer";
}
```
