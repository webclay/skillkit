# Sub-Agents (Facets)

Sub-agents are child agents that run as Durable Object facets co-located inside a parent. Each child gets its own isolated SQLite storage, state, and scheduling. Parents coordinate children through typed RPC.

Official docs: https://developers.cloudflare.com/agents/

## Spawning Sub-Agents

```typescript
import { Agent, callable } from "agents";

export class Orchestrator extends Agent<Env, OrchestratorState> {
  @callable()
  async research(topic: string) {
    // Spawn or retrieve a named child agent
    const researcher = await this.subAgent(Researcher, "research-1");
    const findings = await researcher.analyze(topic);
    return findings;
  }
}

export class Researcher extends Agent<Env, ResearcherState> {
  initialState: ResearcherState = { results: [] };

  async analyze(topic: string) {
    // Each child has its own SQLite, state, and scheduling
    this.sql`INSERT INTO searches (topic) VALUES (${topic})`;
    return { topic, results: ["result1", "result2"] };
  }
}
```

Requirements:
- Child class must extend `Agent` and be exported from the Worker entry point (same file as `routeAgentRequest` or exported via re-export)
- Export name must match class name exactly
- Only the parent needs a Durable Object binding in wrangler.jsonc
- Child classes are discovered automatically via `ctx.exports` - do not add them as top-level DO namespaces

## wrangler.jsonc Configuration

Only the parent agent needs a binding and migration entry. Facet-only child classes must NOT be in `new_sqlite_classes`:

```jsonc
{
  "durable_objects": {
    "bindings": [
      { "name": "Orchestrator", "class_name": "Orchestrator" }
    ]
  },
  "migrations": [
    {
      "tag": "v1",
      "new_sqlite_classes": ["Orchestrator"]
    }
  ]
}
```

If a child class also needs to be used as a top-level agent (not just a facet), add it to both `bindings` and `new_sqlite_classes`.

For test environments using `@cloudflare/vitest-pool-workers`, you may include facet classes as test-only DO bindings without adding them to `new_sqlite_classes`.

## Managing Sub-Agents

```typescript
// Check if a sub-agent exists (synchronous)
const exists = this.hasSubAgent(Researcher, "research-1");

// List all sub-agents
const allAgents = this.listSubAgents();

// List by class
const researchers = this.listSubAgents(Researcher);

// Force stop (storage preserved, restarts on next call)
await this.abortSubAgent(Researcher, "research-1");

// Permanently delete (wipes storage)
await this.deleteSubAgent(Researcher, "research-1");
```

## Parallel Execution

```typescript
@callable()
async parallelResearch(topics: string[]) {
  const results = await Promise.all(
    topics.map(async (topic, i) => {
      const agent = await this.subAgent(Researcher, `research-${i}`);
      return agent.analyze(topic);
    })
  );
  return results;
}
```

## Identity

Inside a sub-agent, `this.name` returns the facet's own name (not the parent's):

```typescript
// In a sub-agent:
this.name        // Facet name (e.g., "research-1")
this.parentPath  // Array of ancestor names (root-first)
this.selfPath    // Full path including self
```

Storage is fully isolated - parent SQL and child SQL are completely separate databases.

## Calling Back to the Parent

Sub-agents can call typed RPC methods on their parent:

```typescript
export class Researcher extends Agent<Env, ResearcherState> {
  async analyze(topic: string) {
    const result = await doWork(topic);

    // Typed RPC back to parent
    const parent = this.parentAgent(Orchestrator);
    await parent.reportComplete(this.name, result);

    return result;
  }
}

export class Orchestrator extends Agent<Env, {}> {
  async reportComplete(childName: string, result: unknown) {
    console.log(`${childName} finished`, result);
  }
}
```

`parentAgent(Cls)` resolves through a root RPC bridge, so facet-only parents (parents that aren't bound as top-level DO namespaces) are also reachable.

## Access Control

Override `onBeforeSubAgent()` on the parent to gate incoming requests routed to sub-agents:

```typescript
async onBeforeSubAgent(_request: Request, { className, name }: SubAgentInfo) {
  if (!this.hasSubAgent(className, name)) {
    return new Response("Not found", { status: 404 });
  }
  // Return void to allow, or return a Response to short-circuit
}
```

## Client Routing to Sub-Agents

Connect directly to a sub-agent from the frontend:

```typescript
import { useAgent } from "agents/react";

const chat = useAgent({
  agent: "Inbox",
  name: userId,
  sub: [{ agent: "Chat", name: chatId }],
});
```

The SDK routes WebSocket connections to the correct child agent via `/agents/{parent}/{name}/sub/{child}/{name}`.

## Scheduling Inside Sub-Agents

Sub-agents can schedule their own tasks. Schedules are scoped to the child - sibling sub-agents cannot cancel each other's schedules.

Use the async APIs only (sync methods throw inside sub-agents):

```typescript
// Inside a sub-agent:
await this.schedule(60, "processData", { itemId: "abc" });
await this.scheduleEvery(300, "checkUpdates", {});
const schedules = await this.listSchedules(); // async - ok
await this.cancelSchedule(scheduleId);

// These throw inside sub-agents (use async alternatives):
// this.getSchedule(id) // THROWS
// this.getSchedules()  // THROWS
```

The physical Durable Object alarm belongs to the top-level parent, but each sub-agent's schedules are isolated. Destroying a sub-agent cleans up its schedules automatically.

## Nested Sub-Agents

Sub-agents can themselves spawn child sub-agents:

```typescript
export class MiddleAgent extends Agent<Env, {}> {
  async run() {
    const child = await this.subAgent(LeafAgent, "leaf-1");
    return child.doWork();
  }
}
```

`subAgent()` throws a descriptive error if the parent class is not exported from the Worker entrypoint. If the class name looks minified (e.g. `_a`), the error includes a bundler-config hint about preserving class names.

## Common Errors

**"Cannot call subAgent from a class that is not bound as a Durable Object"** - The parent class is not exported from the Worker entry point. Add `export { Orchestrator } from "./orchestrator"` to your `server.ts`.

**`this.name` returning the parent's name** - This was a bug in older versions. Make sure you're on `agents >= 0.11.6` and `partyserver >= 0.5.3`.

**Sub-agent schedules not running** - Sub-agents delegate alarms to the parent DO. Ensure the parent stays alive (no premature hibernation) or the parent itself has something keeping it alive.
