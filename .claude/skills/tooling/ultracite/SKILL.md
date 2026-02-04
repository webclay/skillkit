---
name: ultracite
description: Ultracite zero-config linting and formatting for TypeScript. Trigger words - ultracite, lint setup, format code, code quality, eslint config, prettier config
---

# Ultracite

Zero-config linting and formatting for TypeScript projects.

## When to Use This Skill

- Setting up code quality for TypeScript projects
- Formatting code before commits
- Running lint checks in CI/CD
- Replacing ESLint + Prettier with one tool

## When NOT to Use

- Highly customized ESLint configs you can't migrate
- Non-TypeScript/JavaScript projects

## Setup

```bash
npx ultracite@latest init
```

This creates a minimal `biome.json` config:

```json
{
  "$schema": "https://biomejs.dev/schemas/1.9.4/schema.json",
  "extends": ["@ultracite/biome-preset"]
}
```

### Framework Presets

```json
{
  "extends": ["@ultracite/biome-preset-nextjs"]
}
```

Available: `biome-preset`, `biome-preset-nextjs`, `biome-preset-react`, `biome-preset-vue`, `biome-preset-solid`

## Commands

```bash
# Fix all formatting and lint issues
npx ultracite fix

# Check without fixing (for CI)
npx ultracite check

# Diagnose setup issues
npx ultracite doctor

# Watch mode
npx ultracite watch
```

## Package.json Scripts

```json
{
  "scripts": {
    "format": "ultracite fix",
    "format:check": "ultracite check",
    "lint": "ultracite check",
    "lint:fix": "ultracite fix"
  }
}
```

## Git Hooks (Husky + lint-staged)

```bash
npm install --save-dev husky lint-staged
npx husky init
echo "npx lint-staged" > .husky/pre-commit
```

```json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx,json,css,md}": [
      "npx ultracite fix --no-errors-on-unmatched"
    ]
  }
}
```

## CI/CD (GitHub Actions)

```yaml
name: Lint
on: [pull_request, push]
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npx ultracite check
```

## Key Rules Enforced

- **TypeScript**: Strict types, prefer `unknown` over `any`
- **Modern JS**: Arrow functions, `for...of`, optional chaining
- **React**: Function components, hooks at top level, proper keys
- **Security**: No `eval()`, `rel="noopener"` with `target="_blank"`
- **Performance**: No spread in loops, specific imports

## VS Code Setup

`.vscode/settings.json` (auto-generated):
```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "biomejs.biome",
  "editor.codeActionsOnSave": {
    "quickfix.biome": "explicit",
    "source.organizeImports.biome": "explicit"
  }
}
```

Install the Biome VS Code extension.

## Ignoring Files

Create `.biomeignore`:
```
node_modules/
dist/
build/
.next/
*.generated.*
*.d.ts
```

## Migration from ESLint/Prettier

```bash
# Remove old tools
npm uninstall eslint prettier @typescript-eslint/* eslint-*
rm .eslintrc.* .prettierrc.*

# Install Ultracite
npx ultracite@latest init

# Fix everything
npx ultracite fix
```

## Tips

- Run `npx ultracite fix` before every commit
- Configure format-on-save in your editor
- Use the same version in CI and locally

## How to Verify

### Quick Checks
- `npx ultracite check` passes with no errors
- Editor formats on save
- CI pipeline passes lint checks

### Common Issues
- "Command not found": Use `npx ultracite` not just `ultracite`
- Editor not formatting: Install Biome extension, restart editor
- Too many errors: Run `npx ultracite fix` to auto-fix most issues
