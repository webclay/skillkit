---
name: ultracite
description: Linting and formatting with Ultracite. Trigger words - lint, linting, format, type check, typescript error, eslint, biome, oxlint, fix lint, ultracite, code quality
---

# Ultracite

Zero-config linting and formatting for JS/TS projects. Supports three linter backends: **Biome** (recommended), **ESLint + Prettier**, and **Oxlint + Oxfmt**.

## When to Use This Skill

- Before making a commit
- User asks to "check for errors", "lint the code", or "format the code"
- After making significant changes
- User wants to ensure code quality
- Setting up linting in a new project

## Instructions

### Step 0: Determine Runner Command

Check `project/dev-context.md` for the project's `## Package Manager` section to determine the correct runner command. If not specified, check for lock files (`bun.lockb`, `pnpm-lock.yaml`, etc.). Default to `bunx`.

| Package Manager | Runner |
|-----------------|--------|
| bun | bunx |
| pnpm | pnpm dlx |
| npm | npx |
| yarn | yarn dlx |

### Step 1: Detect Active Linter Backend

Check if `ultracite` is in `package.json` devDependencies. Detect the active linter by looking for:

- `biome.jsonc` - Biome
- `eslint.config.mjs` - ESLint
- `.oxlintrc.json` - Oxlint

If none found, Ultracite may not be initialized yet. See **Initialization** below.

### Step 2: Run Ultracite

**Auto-fix issues:**
```bash
[runner] ultracite fix
```

**Check only (read-only, no changes):**
```bash
[runner] ultracite check
```

Both `check` and `fix` accept optional file paths: `[runner] ultracite check src/index.ts`.

### Step 3: Check for Remaining Errors

```bash
[runner] tsc --noEmit
```

### Step 4: Report Results

**If all checks pass:**
```
All checks passed. Ready to commit.
```

**If errors found:**
```
Found X issues:

### TypeScript Errors
- [file:line] - [error message]

### Lint Errors (not auto-fixed)
- [file:line] - [rule] - [message]
```

### Step 5: Fix Remaining Issues

For each error that wasn't auto-fixed:
1. Read the file
2. Understand the error
3. Apply the fix
4. Re-run checks to confirm

## Initialization

If Ultracite is not set up yet, use `[runner] ultracite init` for interactive setup.

For non-interactive (CI) use, pass flags:

```bash
[runner] ultracite init \
  --pm bun \
  --linter biome \
  --editors vscode cursor \
  --agents claude \
  --frameworks react next \
  --integrations husky lint-staged \
  --quiet
```

**Available flags:**

| Flag | Options |
|------|---------|
| `--pm` | `npm`, `yarn`, `pnpm`, `bun` |
| `--linter` | `biome` (recommended), `eslint`, `oxlint` |
| `--editors` | `vscode`, `zed`, `cursor`, `windsurf`, `antigravity`, `kiro`, `trae`, `void` |
| `--agents` | `claude`, `codex`, `copilot`, `cline`, `amp`, `gemini`, `cursor-cli` + more |
| `--frameworks` | `react`, `next`, `solid`, `vue`, `svelte`, `qwik`, `remix`, `angular`, `astro`, `nestjs` |
| `--integrations` | `husky`, `lefthook`, `lint-staged`, `pre-commit` |
| `--hooks` | Enable auto-fix hooks for supported agents/editors |
| `--type-aware` | Enable type-aware linting (oxlint only) |
| `--skip-install` | Skip dependency installation |
| `--quiet` | Suppress prompts (auto-detected when `CI=true`) |

Init creates config that extends Ultracite presets:

```jsonc
// biome.jsonc (Biome backend)
{ "extends": ["ultracite/biome/core", "ultracite/biome/react"] }
```

Framework presets available per linter: `core`, `react`, `next`, `solid`, `vue`, `svelte`, `qwik`, `remix`, `angular`, `astro`, `nestjs`.

## Code Standards

When writing code in a project with Ultracite, follow these standards:

### Formatting

- 2-space indentation
- LF line endings
- 80-character line width
- Semicolons always
- Double quotes (single quotes in JSX)
- ES5 trailing commas
- Arrow parentheses always
- Bracket spacing enabled

### Style

- Arrow functions preferred for callbacks and short functions
- `const` by default, `let` only when reassignment needed, never `var`
- `for...of` over `.forEach()` and indexed `for` loops
- Template literals over string concatenation
- Destructuring for object and array assignments
- Optional chaining (`?.`) and nullish coalescing (`??`) for safer access
- No enums - use objects with `as const`
- No nested ternary operators
- No non-null assertions (`!`)
- Kebab-case filenames enforced
- `useImportType` for type-only imports
- Sorted imports, JSX attributes, interface members, and object properties

### Type Safety

- Explicit types for function parameters and return values when they enhance clarity
- Prefer `unknown` over `any` when the type is genuinely unknown
- Use const assertions (`as const`) for immutable values and literal types
- Leverage TypeScript's type narrowing instead of type assertions
- Meaningful variable names instead of magic numbers

### Async & Promises

- Always `await` promises in async functions
- Use `async/await` syntax instead of promise chains
- Handle errors with try-catch blocks
- Don't use async functions as Promise executors

### React

- Function components only (no class components)
- Call hooks at top level only, never conditionally
- Specify all dependencies in hook dependency arrays correctly
- Use `key` prop for elements in iterables (prefer unique IDs over array indices)
- Don't define components inside other components
- Use semantic HTML and ARIA attributes for accessibility
- React 19+: Use ref as a prop instead of `React.forwardRef`

### Correctness

- No unused imports/variables
- No `console.log`/`debugger`/`alert` in production
- Throw `Error` objects with descriptive messages, not strings
- Use early returns to reduce nesting
- Max cognitive complexity of 20

### Performance

- No accumulating spread in loops
- No barrel files (index files that re-export everything)
- No namespace imports - use specific imports
- Top-level regex literals instead of creating them in loops
- Use proper image components (e.g., Next.js `<Image>`) over `<img>` tags

### Security

- `rel="noopener"` on `target="_blank"` links
- No `dangerouslySetInnerHTML` unless absolutely necessary
- No `eval()` or direct `document.cookie` assignment
- Validate and sanitize user input

### Framework-Specific

- **Next.js:** Use `<Image>` component. Use App Router metadata API for head elements. Use Server Components for async data fetching.
- **Solid/Svelte/Vue/Qwik:** Use `class` and `for` attributes (not `className` or `htmlFor`).

### Overrides (Relaxed Rules)

- **Config files** (`*.config.{js,ts,...}`): Relaxed export and complexity rules
- **Test files** (`*.test.*`, `__tests__/**`): No cognitive complexity limit, `console` and `any` allowed
- **Scripts/bin files**: `console` and `process.env` allowed
- **Storybook files**: Unused variable/import rules relaxed
- **Build output** (`dist/`, `.next/`, `node_modules/`): Formatter and linter disabled entirely

## Troubleshooting

Run `[runner] ultracite doctor` to diagnose setup problems. It checks:

1. Linter installation (biome/eslint/oxlint binary available)
2. Config validity (extends ultracite presets correctly)
3. Ultracite in package.json dependencies
4. Conflicting tools (old `.eslintrc.*`, `.prettierrc.*` files)

**Common fixes:**

- **Conflicting configs:** Delete legacy `.eslintrc.*` and `.prettierrc.*` files after migrating to Ultracite
- **Missing dependency:** Run `[runner] ultracite init` again or manually add `ultracite` to devDependencies
- **Rules not applying:** Ensure config file extends the correct presets for your framework

## Output Format

```
### Summary
- X files passed
- Y files had auto-fixable issues (fixed)
- Z files need manual fixes

### Manual Fixes Needed
[List each issue with file:line and what to fix]

### Files Changed
[List of files modified by auto-fix]
```

## Rules

- Run checks on entire project, not just changed files
- Auto-fix what you can
- Be specific about remaining issues
- Don't change code behavior - only fix style/types
- Report clearly what was fixed vs what needs attention

## How to Verify

- `[runner] ultracite fix` runs without errors
- `[runner] tsc --noEmit` passes
- All auto-fixable issues resolved
