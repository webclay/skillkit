---
name: bun
description: Bun JavaScript runtime and package manager. Trigger words - bun, bun install, bun run, bunx, bun test, bun build
---

# Bun

All-in-one JavaScript runtime, package manager, bundler, and test runner.

## When to Use This Skill

- Faster package installation than npm/pnpm
- Running TypeScript files directly
- Drop-in Node.js replacement
- Bundling and testing

## Key Facts

- **Bun 1.3+** uses `bun.lock` (text-based JSONC) instead of `bun.lockb` (binary)
- Drop-in replacement for Node.js
- Most npm packages work out of the box

## Common Commands

```bash
# Install dependencies
bun install

# Add a package
bun add <package>

# Add dev dependency
bun add -d <package>

# Remove a package
bun remove <package>

# Run scripts from package.json
bun run dev
bun run build
bun run test

# Run a file directly (no compilation needed)
bun run script.ts
```

## Migrating from npm/pnpm

```bash
# 1. Install Bun
curl -fsSL https://bun.sh/install | bash

# 2. Create bun.lock
bun install

# 3. Delete old lockfile
rm pnpm-lock.yaml  # or package-lock.json

# 4. Use bun going forward
bun run dev
```

## Compatibility Notes

- Most Node.js APIs supported (not 100%)
- Some native binding packages may have issues
- Works with Appwrite SDK, Prisma, etc.

## Tips

- Use `bun run` instead of `npm run`
- TypeScript works without compilation
- Significantly faster than npm/pnpm for installs

## How to Verify

### Quick Checks
- `bun --version` shows installed version
- `bun install` completes without errors
- `bun run dev` starts your dev server

### Common Issues
- "Command not found": Restart terminal after install
- Package errors: Check if package has Bun compatibility issues
