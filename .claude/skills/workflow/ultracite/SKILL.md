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

### Step 1: Run Ultracite

```bash
npx ultracite fix
```

This runs:
- ESLint with auto-fix
- Prettier formatting
- TypeScript type checking

### Step 2: Check for Remaining Errors

```bash
npx tsc --noEmit
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

- `npx ultracite fix` runs without errors
- `npx tsc --noEmit` passes
- All auto-fixable issues resolved
