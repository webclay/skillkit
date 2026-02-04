---
name: motia
description: Motia event-driven workflow runtime with Steps. Trigger words - motia, workflow, background job, event-driven, steps, ai agent, orchestration, streaming
---

# Motia

Event-driven runtime for building workflows with Steps - supports TypeScript, Python, Ruby, and AI agents. Includes built-in streaming, state management, and visual debugging.

## When to Use This Skill

- Complex multi-step workflows
- Background job processing
- AI agents with tool calling
- Event-driven microservices
- Need visual workflow debugging (Workbench)
- Real-time streaming updates to clients
- Long-running processes with progress updates

## When NOT to Use

- Simple CRUD APIs (just use your framework's API routes)
- Single-request handlers
- Persistent user data storage (use a database)

## Setup

```bash
npx motia@latest create
cd my-project
bun install
bun run dev
```

Visit `http://localhost:3000` for Workbench UI.

To change the Workbench port (e.g., if Next.js uses 3000):
```json
// package.json
{
  "scripts": {
    "dev": "motia dev --port 3001"
  }
}
```

## Core Concepts

**Steps**: Individual workflow units that subscribe to topics and emit events
**Topics**: Event channels that connect steps
**Flows**: Visual representation of connected steps
**Workbench**: Visual debugger at localhost:3000

## Step Types

### API Step (HTTP Endpoints)

```typescript
// steps/create-order.step.ts
import { ApiRouteConfig, StepHandler } from "motia";
import { z } from "zod";

export const config: ApiRouteConfig = {
  type: "api",
  name: "Create Order",
  path: "/orders",
  method: "POST",
  emits: ["order.created"],
  flows: ["order-flow"],
  // Zod schema for request validation
  bodySchema: z.object({
    items: z.array(z.string()),
    customerId: z.string(),
  }),
};

export const handler: StepHandler<typeof config> = async (req, { emit, logger }) => {
  const { items, customerId } = req.body;
  const orderId = crypto.randomUUID();

  await emit({
    topic: "order.created",
    data: { orderId, items, customerId },
  });

  logger.info("Order created", { orderId });
  return { status: 201, body: { orderId } };
};
```

### Event Step (Background Processing)

```typescript
// steps/process-payment.step.ts
import { EventConfig, StepHandler } from "motia";

export const config: EventConfig = {
  type: "event",
  name: "Process Payment",
  subscribes: ["order.created"],
  emits: ["payment.processed", "payment.failed"],
  flows: ["order-flow"],
};

export const handler: StepHandler<typeof config> = async (input, { emit, logger }) => {
  const { orderId, customerId } = input;

  logger.info("Processing payment", { orderId, customerId });

  try {
    const payment = await processPayment(orderId);

    await emit({
      topic: "payment.processed",
      data: { orderId, paymentId: payment.id },
    });

    logger.info("Payment successful", { paymentId: payment.id });
  } catch (error) {
    await emit({
      topic: "payment.failed",
      data: { orderId, error: error.message },
    });

    logger.error("Payment failed", { orderId, error: error.message });
  }
};
```

### Cron Step (Scheduled Jobs)

```typescript
// steps/daily-report.step.ts
import { CronConfig, StepHandler } from "motia";

export const config: CronConfig = {
  type: "cron",
  name: "Daily Report",
  cron: "0 9 * * *", // 9 AM daily
  emits: ["report.generated"],
};

export const handler: StepHandler<typeof config> = async (_, { emit, logger }) => {
  const report = await generateDailyReport();

  await emit({
    topic: "report.generated",
    data: { report, generatedAt: new Date().toISOString() },
  });
};
```

## State Management

Motia includes a built-in key-value store. Use it for temporary workflow data, NOT persistent user data.

**When to use state:**
- Sharing data between steps (avoids prop drilling through events)
- Temporary workflow data
- API response caching
- Building up results across multiple steps

**When NOT to use state (use a database instead):**
- Persistent user data
- Data that should survive restarts
- Large datasets
- File uploads

```typescript
// Persist data across steps using groups
export const handler: StepHandler<typeof config> = async (input, { state }) => {
  // Save to a group (e.g., "polls" group, keyed by poll ID)
  await state.set("polls", pollId, { question, options });

  // Retrieve from group
  const poll = await state.get<Poll>("polls", pollId);

  // Get all items in a group
  const allPolls = await state.getGroup<Poll>("polls");

  // Delete from group
  await state.delete("polls", pollId);
};
```

## Streaming (Real-time Updates)

Stream workflow progress to clients without setting up WebSockets yourself. Great for long-running processes, live dashboards, or collaborative features.

### 1. Define the stream

```typescript
// streams/workflow-stream.ts
import { z } from "zod";

export const config = {
  name: "workflow-progress",
  schema: z.object({
    stage: z.string(),
    progress: z.number().optional(),
    result: z.any().optional(),
  }),
};
```

### 2. Write to stream from steps

```typescript
// steps/process-vote.step.ts
export const handler: StepHandler<typeof config> = async (input, { streams, logger }) => {
  const { workflowProgress } = streams;

  // Update clients on current stage
  await workflowProgress.set({
    stage: "validating",
    progress: 25,
  });

  // Do validation...

  await workflowProgress.set({
    stage: "applying",
    progress: 75,
  });

  // Apply changes...

  await workflowProgress.set({
    stage: "completed",
    progress: 100,
    result: { success: true },
  });
};
```

### 3. Consume in React client

```bash
bun add @motiadev/stream-client-react
```

```tsx
// app/providers.tsx
import { MotiaStreamProvider } from "@motiadev/stream-client-react";

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <MotiaStreamProvider url={process.env.NEXT_PUBLIC_MOTIA_WS_URL!}>
      {children}
    </MotiaStreamProvider>
  );
}

// components/workflow-status.tsx
import { useStreamItem } from "@motiadev/stream-client-react";

function WorkflowStatus() {
  const { data } = useStreamItem<{ stage: string; progress: number }>("workflow-progress");

  if (!data) return null;

  return (
    <div>
      <p>Stage: {data.stage}</p>
      <progress value={data.progress} max={100} />
    </div>
  );
}
```

### Environment variables for streaming

```env
# .env.local (frontend)
NEXT_PUBLIC_MOTIA_WS_URL=ws://localhost:3001
MOTIA_BASE_URL=http://localhost:3001
```

### Streaming use cases

| Use case | Example |
|----------|---------|
| Long processes | Show validation, processing, completion stages |
| AI responses | Stream tokens as they generate |
| Live dashboards | Real-time metrics updates |
| Collaborative tools | Multiple users see updates instantly |
| Chat/notifications | Push new messages to all connected clients |

## AI Agents with Tools

```typescript
// steps/ai-agent.step.ts
import { EventConfig, StepHandler } from "motia";
import Anthropic from "@anthropic-ai/sdk";

export const config: EventConfig = {
  type: "event",
  name: "AI Agent",
  subscribes: ["user.question"],
  emits: ["agent.response"],
};

const tools = [
  {
    name: "search_orders",
    description: "Search customer orders",
    input_schema: {
      type: "object",
      properties: {
        customerId: { type: "string" },
      },
      required: ["customerId"],
    },
  },
];

export const handler: StepHandler<typeof config> = async (input, { emit }) => {
  const client = new Anthropic();

  const response = await client.messages.create({
    model: "claude-sonnet-4-20250514",
    max_tokens: 1024,
    tools,
    messages: [{ role: "user", content: input.question }],
  });

  // Handle tool calls
  for (const block of response.content) {
    if (block.type === "tool_use" && block.name === "search_orders") {
      const orders = await searchOrders(block.input.customerId);
      // Continue conversation with tool result...
    }
  }

  await emit({
    topic: "agent.response",
    data: { response: response.content },
  });
};
```

## Project Structure

```
my-project/
├── steps/
│   ├── api/
│   │   └── create-order.step.ts
│   ├── events/
│   │   └── process-payment.step.ts
│   └── cron/
│       └── daily-report.step.ts
├── motia.config.ts
└── package.json
```

## Configuration

```typescript
// motia.config.ts
import { MotiaConfig } from "motia";

const config: MotiaConfig = {
  state: {
    adapter: "redis",
    redis: { url: process.env.REDIS_URL },
  },
};

export default config;
```

## Frontend Integration

Motia is your backend - connect it to any frontend framework (TanStack Start, Next.js, etc.).

### Project structure

```
my-project/
├── motia/              # Motia backend
│   ├── steps/
│   ├── streams/
│   └── motia.config.ts
└── frontend/           # TanStack Start, Next.js, etc.
    ├── app/
    └── package.json
```

### Making API calls to Motia

```typescript
// frontend/lib/motia.ts
const MOTIA_BASE_URL = process.env.MOTIA_BASE_URL || "http://localhost:3001";

export async function createPoll(data: { question: string; options: string[] }) {
  const response = await fetch(`${MOTIA_BASE_URL}/polls`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(data),
  });
  return response.json();
}

export async function getPolls() {
  const response = await fetch(`${MOTIA_BASE_URL}/polls`);
  return response.json();
}
```

### Server-side calls (recommended)

Make Motia calls from your server (API routes, server functions) to avoid CORS issues:

```typescript
// TanStack Start server function
import { createServerFn } from "@tanstack/react-start";

export const createPoll = createServerFn({ method: "POST" })
  .validator((data: { question: string; options: string[] }) => data)
  .handler(async ({ data }) => {
    const response = await fetch(`${process.env.MOTIA_BASE_URL}/polls`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(data),
    });
    return response.json();
  });
```

## Context Object

The handler receives a context object with these utilities:

| Property | Purpose |
|----------|---------|
| `emit` | Emit events for other steps to subscribe to |
| `logger` | Structured logging (integrated with Workbench) |
| `state` | Key-value store for workflow data |
| `streams` | Write real-time updates to clients |
| `traceId` | Unique ID linking all steps in one workflow run |

```typescript
export const handler: StepHandler<typeof config> = async (input, context) => {
  const { emit, logger, state, streams, traceId } = context;

  logger.info("Processing started", { traceId });
  // Use context utilities...
};
```

## Tips

- Use Workbench to visualize and debug flows
- Keep steps small and focused (like React components)
- Use `flows` in config to organize related steps
- Topics follow `entity.action` naming (order.created, vote.validated)
- Use `logger` instead of `console.log` (shows in Workbench)
- State is for workflow data, not persistent storage
- Run Motia on a different port if your frontend uses 3000

## Free Tier Limits

- Projects: 3
- Environments: 3
- Deployments: 6

If you reach a limit, just remove a previous deployment to create a new one.

## How to Verify

### Quick Checks
- `bun run dev` starts without errors
- Workbench shows your flows at localhost:3000
- API steps respond to HTTP requests (test from Workbench Endpoints tab)
- Events flow between steps (visible in Workbench)
- Logs appear in Workbench Logs tab

### Common Issues
- Step not appearing: Check file ends with `.step.ts`
- Events not flowing: Verify topic names match exactly
- State not persisting: Check Redis connection if using Redis adapter
- CORS errors: Make API calls from your server, not browser directly
- Port conflict: Change Motia port with `--port 3001` flag
