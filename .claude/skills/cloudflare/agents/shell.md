# @cloudflare/shell

Sandboxed JavaScript execution and filesystem runtime for Cloudflare Workers agents. Provides a virtual filesystem (in-memory or durable SQLite-backed) and git operations - designed for agent workflows that need structured state operations.

> **Not a bash interpreter.** Does not parse shell syntax or emulate POSIX behavior. Executes JavaScript.

> **Experimental** - API surface is still settling, expect breaking changes.

GitHub: https://github.com/cloudflare/agents/tree/main/packages/shell

## Install

```bash
npm install @cloudflare/shell @cloudflare/codemode agents
```

## What It Provides

- `Workspace` - durable file storage backed by SQLite + optional R2
- `InMemoryFs` / `WorkspaceFileSystem` - `FileSystem` implementations
- `stateTools(workspace)` - `ToolProvider` for codemode that exposes `state.*` in sandboxed executions
- `createGit(filesystem)` - pure-JS git operations via isomorphic-git
- `gitTools(workspace)` - `ToolProvider` for codemode that exposes `git.*` with auto-injected auth
- A prebuilt `state` stdlib with type declarations for LLM prompts

## Durable Workspace in an Agent

```typescript
import { Agent } from "agents";
import { Workspace } from "@cloudflare/shell";
import { stateTools } from "@cloudflare/shell/workers";
import { DynamicWorkerExecutor, resolveProvider } from "@cloudflare/codemode";

class MyAgent extends Agent<Env> {
  workspace = new Workspace({
    sql: this.ctx.storage.sql,
    r2: this.env.MY_BUCKET,     // Optional R2 for large files
    name: () => this.name,       // Namespace by agent instance
  });

  async run(code: string) {
    const executor = new DynamicWorkerExecutor({ loader: this.env.LOADER });
    return executor.execute(code, [
      resolveProvider(stateTools(this.workspace)),
    ]);
  }
}
```

The `state` object in the sandbox exposes:

```js
// Inside sandboxed code:
await state.readFile("/src/app.ts")
await state.writeFile("/src/app.ts", content)
await state.glob("/src/**/*.ts")
await state.diff("/src/app.ts", newContent)
await state.writeJson("/data.json", obj)
await state.readJson("/data.json")
await state.deleteFile("/tmp/scratch.ts")
await state.listDir("/src")
await state.exists("/src/app.ts")
```

## In-Memory Filesystem

For ephemeral work that doesn't need to persist:

```typescript
import { createMemoryStateBackend } from "@cloudflare/shell";
import { stateToolsFromBackend } from "@cloudflare/shell/workers";
import { DynamicWorkerExecutor, resolveProvider } from "@cloudflare/codemode";

const backend = createMemoryStateBackend({
  files: {
    "/src/app.ts": 'export const answer = "foo";\n',
  },
});

const executor = new DynamicWorkerExecutor({ loader: env.LOADER });

const result = await executor.execute(
  `async () => {
    const text = await state.readFile("/src/app.ts");
    await state.writeFile("/src/app.ts", text.replace("foo", "bar"));
    return await state.readFile("/src/app.ts");
  }`,
  [resolveProvider(stateToolsFromBackend(backend))]
);
```

## Git Operations

Clone, commit, push repositories using pure-JS git backed by the virtual filesystem:

```typescript
import { Workspace, WorkspaceFileSystem } from "@cloudflare/shell";
import { createGit } from "@cloudflare/shell/git";

class MyAgent extends Agent<Env> {
  workspace = new Workspace({
    sql: this.ctx.storage.sql,
    name: () => this.name,
  });

  async cloneAndModify() {
    const git = createGit(new WorkspaceFileSystem(this.workspace));

    await git.clone({ url: "https://github.com/org/repo", depth: 1 });
    await this.workspace.writeFile("/README.md", "# Updated");
    await git.add({ filepath: "." });
    await git.commit({
      message: "update readme",
      author: { name: "Agent", email: "agent@example.com" },
    });
    await git.push({ token: this.env.GITHUB_TOKEN });
  }
}
```

### Git Commands

`init`, `clone`, `status`, `add`, `rm`, `commit`, `log`, `branch`, `checkout`, `fetch`, `pull`, `push`, `diff`, `remote`

## Git Tools for Codemode

Expose git commands to sandboxed LLM code with auto-injected auth (secrets never visible to the LLM):

```typescript
import { stateTools } from "@cloudflare/shell/workers";
import { gitTools } from "@cloudflare/shell/git";

const providers = [
  resolveProvider(stateTools(this.workspace)),
  resolveProvider(gitTools(this.workspace, { token: this.env.GITHUB_TOKEN })),
];

// LLM code can now use:
// await git.clone({ url: "..." })
// await git.commit({ message: "..." })
// await git.push({})
```

For Basic auth servers:

```typescript
resolveProvider(
  gitTools(this.workspace, {
    auth: { username: "git", password: this.env.GIT_PASSWORD },
  })
)
```

Direct tool-call auth takes precedence over defaults.

## With Think

Think's built-in workspace already uses `@cloudflare/shell` under the hood. You get `state.*` tools automatically in every Think agent. Override the workspace to customize:

```typescript
import { Think } from "@cloudflare/think";
import { Workspace } from "@cloudflare/shell";

export class MyAgent extends Think<Env> {
  override workspace = new Workspace({
    sql: this.ctx.storage.sql,
    r2: this.env.R2,
    name: () => this.name,
  });
}
```

For custom tool factories:

```typescript
import { createWorkspaceTools } from "@cloudflare/think/tools/workspace";

const tools = createWorkspaceTools(myCustomStorage);
```
