---
name: ultracite
description: Linting and TypeScript checking with Ultracite. Trigger words - lint, linting, type check, typescript error, eslint, fix lint, ultracite
---

# Ultracite

Runs linting and TypeScript validation before commits.

## When to Use This Skill

- Before making a commit
- User asks to "check for errors" or "lint the code"
- After making significant changes
- User wants to ensure code quality

## Instructions

### Step 0: Determine Runner Command

Check `project/dev-context.md` for the project's `## Package Manager` section to determine the correct runner command. If not specified, check for lock files (`bun.lockb`, `pnpm-lock.yaml`, etc.). Default to `bunx`.

| Package Manager | Runner |
|-----------------|--------|
| bun | bunx |
| pnpm | pnpm dlx |
| npm | npx |
| yarn | yarn dlx |

### Step 1: Run Ultracite

```bash
[runner] ultracite fix
```

This runs:
- ESLint with auto-fix
- Prettier formatting
- TypeScript type checking

### Step 2: Check for Remaining Errors

```bash
[runner] tsc --noEmit
```

### Step 3: Report Results

**If all checks pass:**
```
All checks passed. Ready to commit.
```

**If errors found:**
```
Found X issues:

### TypeScript Errors
- [file:line] - [error message]

### ESLint Errors (not auto-fixed)
- [file:line] - [rule] - [message]
```

### Step 4: Fix Remaining Issues

For each error that wasn't auto-fixed:
1. Read the file
2. Understand the error
3. Apply the fix
4. Re-run checks to confirm

## Common Fixes

### TypeScript
- Add missing types
- Fix type mismatches
- Remove unused variables
- Add null checks

### ESLint
- Fix import order
- Remove unused imports
- Fix naming conventions
- Add missing dependencies to useEffect

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
- Don't change code behavior — only fix style/types
- Report clearly what was fixed vs what needs attention

## How to Verify

- `[runner] ultracite fix` runs without errors
- `[runner] tsc --noEmit` passes
- All auto-fixable issues resolved
