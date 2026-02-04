---
name: update-stack
description: Update all project dependencies safely. Trigger words - update stack, update dependencies, upgrade packages, update frameworks, update all, upgrade stack, npm update, bun update
---

# Update Stack

Safely updates all project dependencies to their latest compatible versions, checks for breaking changes, and fixes any issues that arise.

## When to Use This Skill

- User says "update stack", "update dependencies", or "upgrade packages"
- User wants to update to latest versions of frameworks
- User asks about outdated packages
- Periodic maintenance (recommended: monthly)

## Instructions

### Step 1: Detect Package Manager

Check for lock files:

| Lock File | Package Manager | Update Command |
|-----------|-----------------|----------------|
| `bun.lockb` | bun | `bun update` |
| `pnpm-lock.yaml` | pnpm | `pnpm update` |
| `yarn.lock` | yarn | `yarn upgrade` |
| `package-lock.json` | npm | `npm update` |

### Step 2: Check Current State

Before updating, analyze the project:

1. **Read package.json** - List current dependencies and versions
2. **Check for outdated packages:**

```bash
# For bun
bun outdated

# For pnpm
pnpm outdated

# For npm
npm outdated

# For yarn
yarn outdated
```

3. **Identify major version updates** - These may have breaking changes

### Step 3: Categorize Updates

Present updates to user in categories:

```
I found the following updates available:

**Safe updates (patch/minor - no breaking changes expected):**
- @tanstack/react-query: 5.90.0 → 5.92.0
- tailwindcss: 4.0.6 → 4.0.8
- drizzle-orm: 0.45.0 → 0.45.2

**Major updates (may have breaking changes - review recommended):**
- better-auth: 1.4.12 → 2.0.0 ⚠️
- @tanstack/react-start: 1.157.17 → 2.0.0 ⚠️

**Recommendations:**
- Safe updates: Apply all
- Major updates: Review changelog before updating

How would you like to proceed?
A. Update safe packages only (recommended)
B. Update everything including major versions
C. Let me pick specific packages
```

### Step 4: Perform Updates

Based on user choice:

**For safe updates only:**
```bash
# Bun
bun update --latest

# pnpm (updates within semver range)
pnpm update

# npm
npm update

# yarn
yarn upgrade
```

**For specific major version updates:**
```bash
# Bun
bun add package-name@latest

# pnpm
pnpm add package-name@latest

# npm
npm install package-name@latest

# yarn
yarn add package-name@latest
```

### Step 5: Verify Updates

After updating:

1. **Check for TypeScript errors:**
```bash
# Run type check
bun run typecheck
# or
pnpm typecheck
# or
npx tsc --noEmit
```

2. **Run the linter:**
```bash
npx ultracite fix
```

3. **Start the dev server:**
```bash
bun dev
# or
pnpm dev
```

4. **Run tests (if they exist):**
```bash
bun test
# or
pnpm test
```

### Step 6: Handle Breaking Changes

If errors occur after updating:

1. **Read the error message** - Identify which package caused it
2. **Check the changelog** - Look for migration guides:
   - TanStack: https://tanstack.com/router/latest/docs/framework/react/guide/migrating
   - Better Auth: https://www.better-auth.com/docs/changelog
   - Drizzle: https://orm.drizzle.team/docs/changelog
   - Supabase: https://supabase.com/docs/guides/resources/migrating

3. **Apply fixes** - Update code to match new API
4. **If stuck, rollback:**
```bash
# Restore lock file from git
git checkout -- bun.lockb  # or pnpm-lock.yaml, etc.

# Reinstall
bun install
```

### Step 7: Report Results

After successful update:

```
Stack updated successfully!

**Updated packages:**
- @tanstack/react-query: 5.90.0 → 5.92.0
- tailwindcss: 4.0.6 → 4.0.8
- [... list all updated packages ...]

**Verification:**
- TypeScript: No errors
- Linter: Passed
- Dev server: Running

**Skipped (major version - manual review needed):**
- better-auth: 1.4.12 → 2.0.0

Your project is up to date!
```

## Common Package-Specific Notes

### TanStack Start / Router
- Check router migration guide for major versions
- Route definitions may change between versions
- `beforeLoad` API may change

### Better Auth
- Session handling may change
- Check for new required config options
- Plugin APIs may be updated

### Drizzle
- Schema syntax may change
- Migration commands may differ
- Check for deprecated methods

### Supabase
- Client initialization may change
- RLS policy syntax updates
- Realtime API changes

### Tailwind CSS
- v4 has significant changes from v3
- Config file format changed
- Some utility classes renamed

## Automated Update Script

For users who want to run this regularly, suggest adding to package.json:

```json
{
  "scripts": {
    "update:check": "bun outdated",
    "update:safe": "bun update",
    "update:all": "bun update --latest"
  }
}
```

## Tips

- Update regularly (monthly) to avoid large jumps
- Always commit before updating so you can rollback
- Read changelogs for major versions before updating
- Test the app after updates before deploying
- Keep lock files in git for reproducible builds

## When NOT to Update

- Right before a deadline/launch
- If you don't have time to fix potential issues
- During active feature development (finish first, then update)
- If a major version just released (wait a week for hotfixes)
