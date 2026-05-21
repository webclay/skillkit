# Managed Fiber Jobs

Managed fibers are durable background jobs with idempotent acceptance, optional completion waiting, inspection, cancellation, explicit recovery outcomes, and retained terminal status records.

Available from `agents` v0.13.1. Works inside sub-agents as of v0.12.0.

Official docs: https://developers.cloudflare.com/agents/

## When to Use Fibers vs Schedules

| Use fibers when... | Use schedules when... |
|---|---|
| Work is triggered by an external event (webhook, API call) | Work runs on a time-based pattern |
| You need idempotent acceptance (same job ID = same fiber) | You need cron or interval execution |
| You want to wait for completion or inspect status | The job fires and you don't need to track it |
| Recovery and retry semantics matter | Simple one-shot or recurring callbacks |

## Basic Usage

### Starting a Fiber

```typescript
import { Agent } from "agents";

export class WorkerAgent extends Agent<Env, {}> {
  async onRequest(request: Request): Promise<Response> {
    const { jobId, payload } = await request.json();

    const fiber = await this.runFiber(jobId, async () => {
      // Work that runs durably
      const result = await processPayload(payload);
      await this.env.DB.insert("results", result);
      return { success: true, result };
    });

    return Response.json({ fiberId: fiber.id, status: fiber.status });
  }
}
```

`runFiber(id, fn)` accepts an idempotency key as the first argument. Calling `runFiber` with the same `id` while the fiber is still running returns the existing fiber instead of starting a duplicate.

### Waiting for Completion

```typescript
const fiber = await this.runFiber(jobId, async () => {
  return await doLongWork();
});

// Wait for the fiber to finish (returns when it completes or throws on failure)
const result = await fiber.wait();
```

### Keeping the Agent Alive During Fibers

By default, an agent hibernates when idle. Use `keepAlive()` to keep the Durable Object active while a fiber is running:

```typescript
async onStart() {
  await this.keepAlive();
}
```

Or keep alive only while a specific condition holds:

```typescript
await this.keepAliveWhile(async () => {
  const active = await this.listFibers({ status: "running" });
  return active.length > 0;
});
```

## Inspecting Fibers

```typescript
// Get a specific fiber by ID
const fiber = await this.getFiber(fiberId);
console.log(fiber.status); // "running" | "completed" | "failed" | "cancelled"
console.log(fiber.result); // result value if completed
console.log(fiber.error);  // error message if failed

// List fibers
const running = await this.listFibers({ status: "running" });
const all = await this.listFibers();
```

Terminal status records are retained so you can inspect completed/failed/cancelled fibers after they finish.

## Cancelling a Fiber

```typescript
await this.cancelFiber(fiberId);
```

Cancelling sends an abort signal into the running fiber. Your fiber function should handle `AbortError` if it needs to clean up:

```typescript
const fiber = await this.runFiber(jobId, async (signal) => {
  const result = await longRunningTask(signal);
  return result;
});
```

## Recovery and Explicit Outcomes

Think's `chatRecovery` uses managed fibers under the hood - each chat turn is wrapped in a fiber so interrupted turns can be recovered and resumed. You can use the same pattern for your own application-owned jobs.

Explicit recovery outcomes let you control what happens when a fiber is recovered after a Durable Object restart:

```typescript
const fiber = await this.runFiber(jobId, async () => {
  const checkpoint = await loadCheckpoint();
  if (checkpoint) {
    return await continueFrom(checkpoint);
  }
  return await startFresh();
});
```

## Sub-Agent Support

Fibers work inside sub-agents since v0.12.0. The physical Durable Object alarm is delegated to the top-level parent, while the fiber runs in the owning sub-agent:

```typescript
export class WorkerAgent extends Agent<Env, {}> {
  async processJob(jobId: string, payload: unknown) {
    const fiber = await this.runFiber(jobId, async () => {
      return await doWork(payload);
    });
    return fiber;
  }
}

export class Orchestrator extends Agent<Env, {}> {
  async dispatch(jobId: string, payload: unknown) {
    const worker = await this.subAgent(WorkerAgent, `worker-${jobId}`);
    return worker.processJob(jobId, payload);
  }
}
```

## Fiber vs Think's submitMessages()

`submitMessages()` is Think's conversation-level admission API. It owns the chat turn queue, idempotent retry, and status. Use `submitMessages()` for AI chat turns; use `runFiber()` for application-owned jobs around those turns (accepting a webhook, restoring provider state, posting a visible reply).

The two compose: `chatRecovery: true` on a Think agent wraps each `submitMessages()` call in `runFiber()` automatically.
