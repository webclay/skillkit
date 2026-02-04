---
name: workflow-devkit
description: Vercel AI SDK Workflow DevKit for durable, resumable workflows. Trigger words - workflow devkit, durable workflow, resumable task, ai pipeline, long-running task, workflow step
---

# Workflow DevKit

Vercel's library for building durable, multi-step workflows that survive failures and server restarts.

## When to Use This Skill

- Multi-step AI pipelines (research → analyze → summarize)
- Long-running tasks that might timeout
- Workflows needing human-in-the-loop approval
- Tasks that must resume after failures
- Background job processing with Vercel

## When NOT to Use

- Simple single-request handlers
- Real-time streaming responses
- Sub-second response requirements

## Setup

```bash
pnpm add ai @ai-sdk/openai @upstash/workflow
```

### Environment Variables

```env
QSTASH_TOKEN=...           # From Upstash Console
OPENAI_API_KEY=sk-...      # Or other AI provider
```

## Basic Workflow

```typescript
// app/api/workflow/route.ts
import { serve } from "@upstash/workflow/nextjs";
import { generateText } from "ai";
import { openai } from "@ai-sdk/openai";

export const { POST } = serve(async (context) => {
  // Step 1: Research
  const research = await context.run("research", async () => {
    const { text } = await generateText({
      model: openai("gpt-4-turbo"),
      prompt: `Research: ${context.requestPayload.topic}`,
    });
    return text;
  });

  // Step 2: Analyze
  const analysis = await context.run("analyze", async () => {
    const { text } = await generateText({
      model: openai("gpt-4-turbo"),
      prompt: `Analyze this research: ${research}`,
    });
    return text;
  });

  // Step 3: Generate Report
  const report = await context.run("generate-report", async () => {
    const { text } = await generateText({
      model: openai("gpt-4-turbo"),
      prompt: `Create a report from: ${analysis}`,
    });
    return text;
  });

  return { report };
});
```

## Workflow Patterns

### Parallel Steps

```typescript
export const { POST } = serve(async (context) => {
  // Run multiple steps in parallel
  const [userProfile, orderHistory, recommendations] = await Promise.all([
    context.run("fetch-profile", () => fetchProfile(context.requestPayload.userId)),
    context.run("fetch-orders", () => fetchOrders(context.requestPayload.userId)),
    context.run("get-recommendations", () => getRecommendations(context.requestPayload.userId)),
  ]);

  return { userProfile, orderHistory, recommendations };
});
```

### Delayed Execution

```typescript
export const { POST } = serve(async (context) => {
  const order = await context.run("create-order", () => createOrder(context.requestPayload));

  // Wait 24 hours before follow-up
  await context.sleep("wait-for-delivery", 60 * 60 * 24);

  await context.run("send-feedback-request", () => sendFeedbackEmail(order.id));
});
```

### Human-in-the-Loop

```typescript
export const { POST } = serve(async (context) => {
  const draft = await context.run("generate-draft", async () => {
    const { text } = await generateText({
      model: openai("gpt-4-turbo"),
      prompt: "Write a blog post about AI",
    });
    return text;
  });

  // Notify human and wait for approval
  await context.run("request-approval", () => sendApprovalRequest(draft));

  // Wait for external event (human approval)
  const { approved, feedback } = await context.waitForEvent("approval-response", {
    timeout: "7d",
  });

  if (!approved) {
    // Revise based on feedback
    return await context.run("revise", () => reviseDraft(draft, feedback));
  }

  return await context.run("publish", () => publishPost(draft));
});
```

### Error Handling & Retries

```typescript
export const { POST } = serve(async (context) => {
  const result = await context.run(
    "flaky-api-call",
    async () => {
      const response = await fetch("https://api.example.com/data");
      if (!response.ok) throw new Error("API failed");
      return response.json();
    },
    {
      retries: 3,
      backoff: "exponential", // 1s, 2s, 4s
    }
  );

  return result;
});
```

### Conditional Branching

```typescript
export const { POST } = serve(async (context) => {
  const sentiment = await context.run("analyze-sentiment", () =>
    analyzeSentiment(context.requestPayload.message)
  );

  if (sentiment === "negative") {
    await context.run("escalate", () => escalateToHuman(context.requestPayload));
    return { status: "escalated" };
  }

  const response = await context.run("auto-respond", () =>
    generateAutoResponse(context.requestPayload.message)
  );

  return { status: "resolved", response };
});
```

## Triggering Workflows

```typescript
// From another API route or server action
const response = await fetch("https://your-app.vercel.app/api/workflow", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ topic: "AI in Healthcare" }),
});
```

### Send External Events

```typescript
// For human-in-the-loop workflows
import { Client } from "@upstash/workflow";

const client = new Client({ token: process.env.QSTASH_TOKEN });

await client.notify({
  eventId: "approval-response",
  eventData: { approved: true },
});
```

## Project Structure

```
app/
├── api/
│   ├── workflow/
│   │   └── route.ts        # Main workflow endpoint
│   ├── research/
│   │   └── route.ts        # Research workflow
│   └── content/
│       └── route.ts        # Content generation workflow
```

## Tips

- Each `context.run()` step is durable - survives restarts
- Steps should be idempotent (safe to retry)
- Use `context.sleep()` for delays instead of setTimeout
- Workflow state persists in Upstash
- Monitor workflows in Upstash Console

## How to Verify

### Quick Checks
- Workflow endpoint returns 200 on POST
- Check Upstash Console for workflow execution
- Verify steps execute in order

### Common Issues
- "QSTASH_TOKEN not found": Set environment variable
- Workflow not resuming: Ensure steps are idempotent
- Timeout errors: Break into smaller steps
- Steps running twice: Add idempotency keys
