# @cloudflare/codemode

Code Mode lets LLMs write executable JavaScript code to orchestrate multiple operations, instead of making individual tool calls. LLMs are better at writing code than calling tools - they've seen millions of lines of real-world code but only contrived tool-calling examples.

Code Mode generates TypeScript type definitions from your tools for LLM context, and executes the generated JavaScript in secure, isolated sandboxes with millisecond startup times.

> **Experimental** - may have breaking changes. Use with caution in production.

GitHub: https://github.com/cloudflare/agents/tree/main/packages/codemode

## Install

```bash
# With Vercel AI SDK
npm install @cloudflare/codemode agents ai zod

# Utilities only (no framework peer deps)
npm install @cloudflare/codemode
```

## wrangler.jsonc

Requires a `worker_loaders` binding to run code in isolated sandboxes:

```jsonc
{
  "worker_loaders": [
    {
      "binding": "LOADER"
    }
  ]
}
```

## Quick Start

`createCodeTool` takes your tools and returns a single AI SDK tool. The LLM writes code that calls your tools:

```typescript
import { createCodeTool } from "@cloudflare/codemode/ai";
import { DynamicWorkerExecutor } from "@cloudflare/codemode";
import { streamText, tool } from "ai";
import { z } from "zod";

const tools = {
  getWeather: tool({
    description: "Get weather for a location",
    inputSchema: z.object({ location: z.string() }),
    execute: async ({ location }) => `Weather in ${location}: 72°F, sunny`,
  }),
  sendEmail: tool({
    description: "Send an email",
    inputSchema: z.object({ to: z.string(), subject: z.string(), body: z.string() }),
    execute: async ({ to, subject, body }) => `Email sent to ${to}`,
  }),
};

const executor = new DynamicWorkerExecutor({ loader: env.LOADER });
const codemode = createCodeTool({ tools, executor });

const result = streamText({
  model,
  system: "You are a helpful assistant.",
  messages,
  tools: { codemode },
});
```

The LLM generates code like:

```js
async () => {
  const weather = await codemode.getWeather({ location: "London" });
  if (weather.includes("sunny")) {
    await codemode.sendEmail({
      to: "team@example.com",
      subject: "Nice day!",
      body: `It's ${weather}`,
    });
  }
  return { weather, notified: true };
};
```

## Tool Providers

Tool providers compose capabilities from different packages into a single sandbox. Each provider contributes tools under a namespace:

```typescript
import { createCodeTool } from "@cloudflare/codemode/ai";
import { DynamicWorkerExecutor } from "@cloudflare/codemode";
import { stateTools } from "@cloudflare/shell/workers";

const executor = new DynamicWorkerExecutor({ loader: env.LOADER });

const codemode = createCodeTool({
  tools: [
    { tools: myTools },         // codemode.myTool({ query: "test" })
    stateTools(workspace),      // state.readFile("/path")
  ],
  executor,
});
```

The LLM can use both namespaces in the same code block:

```js
async () => {
  const files = await state.glob("/src/**/*.ts");
  const results = await Promise.all(
    files.map((f) => codemode.analyzeFile({ path: f }))
  );
  await state.writeJson("/report.json", results);
  return results.length;
};
```

## TanStack AI

```typescript
import { createCodeTool, tanstackTools } from "@cloudflare/codemode/tanstack-ai";
import { DynamicWorkerExecutor } from "@cloudflare/codemode";
import { toolDefinition } from "@tanstack/ai";
import { z } from "zod";

const getWeather = toolDefinition({
  name: "get_weather",
  description: "Get weather for a location",
  inputSchema: z.object({ location: z.string() }),
}).server(async ({ location }) => `Weather in ${location}: 72°F, sunny`);

const executor = new DynamicWorkerExecutor({ loader: env.LOADER });
const codeTool = createCodeTool({
  tools: [tanstackTools([getWeather])],
  executor,
});
```

## Custom ToolProvider

Any package can provide tools to the sandbox by implementing `ToolProvider`:

```typescript
import type { ToolProvider } from "@cloudflare/codemode";

const dbProvider: ToolProvider = {
  name: "db",                    // Namespace in sandbox: db.query(...)
  tools: {
    query: {
      description: "Run a SQL query",
      execute: async (sql: unknown) => db.prepare(sql as string).all(),
    },
  },
  positionalArgs: true,          // Enables db.query("SELECT ...") instead of db.query({ sql: "..." })
};

const codemode = createCodeTool({ tools: [dbProvider], executor });
```

## Direct Executor Use

Execute code directly without the tool wrapper:

```typescript
import { DynamicWorkerExecutor, resolveProvider } from "@cloudflare/codemode";

const executor = new DynamicWorkerExecutor({ loader: env.LOADER });

const result = await executor.execute(
  `async () => {
    const data = await codemode.fetchData({ id: "123" });
    return data.value * 2;
  }`,
  [resolveProvider(myProvider)]
);
```

## Browser Tools

Codemode ships a browser tool provider using Cloudflare Browser Rendering:

```typescript
import { createBrowserTools } from "@cloudflare/codemode";

// wrangler.jsonc: { "browser": { "binding": "BROWSER" } }
const browserTools = createBrowserTools({
  browser: env.BROWSER,
  loader: env.LOADER,
});
```

The browser tools let the LLM write code to inspect DOM, take screenshots, scrape content, and debug frontend issues.

## Use With Think

Think has a built-in code execution tool for easy integration:

```typescript
import { Think } from "@cloudflare/think";
import { createExecuteTool } from "@cloudflare/think/tools/execute";

export class MyAgent extends Think<Env> {
  getModel() { ... }

  getTools() {
    return {
      execute: createExecuteTool({ tools: wsTools, loader: this.env.LOADER }),
    };
  }
}
```

## Sandbox Security

- Code runs in an isolated Cloudflare Worker with millisecond startup
- Outbound network is blocked by default
- Sandbox has a configurable execution timeout
- No access to the parent Worker's bindings or globals
- Secrets passed via tool providers are never exposed to the generated code
