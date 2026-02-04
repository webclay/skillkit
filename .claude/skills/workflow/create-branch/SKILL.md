---
name: create-branch
description: Auto-create a feature branch from the current task. Trigger words - branch, create branch, new branch, start task
---

# Create Branch

Automatically creates a feature branch based on your current task in `tasks.md`. The branch name is generated from the task description with proper prefixes (feature/, fix/, etc.).

## When to Use This Skill

- User says "branch", "create branch", or "start task"
- Beginning work on a new task
- Need to switch from main/dev to a feature branch

## Workflow

### Step 1: Check Current Branch

```bash
git branch --show-current
```

If already on a feature branch (not main/dev/master), ask if they want to:
- Stay on current branch
- Create a new branch anyway
- Switch to a different existing branch

### Step 2: Find Current Task

Read `project/tasks.md` to find the task marked as "in progress" or the highest priority "pending" task.

**If no task found:**
```
No task found in tasks.md.

Would you like to:
1. Create a branch manually (provide a name)
2. Check task status first (/tasks)
```

**If task found:**
Show the task and suggest branch name based on it.

### Step 3: Determine Branch Type

Analyze the task description to determine branch prefix:

**Keywords → Prefix:**
- "add", "create", "build", "implement", "new" → `feature/`
- "fix", "bug", "error", "broken" → `fix/`
- "update", "improve", "enhance", "modify" → `update/`
- "refactor", "restructure", "reorganize" → `refactor/`
- "optimize", "performance", "speed up" → `optimize/`
- "security", "vulnerability", "secure" → `security/`
- Default → `feature/`

### Step 4: Generate Branch Name

From task name: "Add user authentication"
1. Remove common words: "Add", "Create", "Implement", "Fix"
2. Convert to lowercase
3. Replace spaces with hyphens
4. Remove special characters
5. Limit to 50 characters

**Examples:**
- "Add user authentication" → `feature/user-authentication`
- "Fix login redirect bug" → `fix/login-redirect-bug`
- "Update dashboard layout" → `update/dashboard-layout`
- "Refactor API calls" → `refactor/api-calls`

### Step 5: Check for Uncommitted Changes

```bash
git status --porcelain
```

**If changes exist:**
```
You have uncommitted changes:
- [list changed files]

What would you like to do?
1. Commit changes first (recommended)
2. Stash changes and create branch
3. Create branch anyway (changes come with you)
```

### Step 6: Create and Switch to Branch

```bash
git checkout -b feature/branch-name
```

### Step 7: Confirm Creation

```
Branch created and checked out!

Branch: feature/user-authentication
Based on: dev (or main)
Task: Add user authentication

Ready to start working. When done:
1. /log - Update changelog and tasks
2. /push - Push to GitHub and create PR
3. /merge - Merge PR after approval
```

## Examples

**Starting a new task:**

```
User: "/branch"

Claude: "Looking at your tasks...

Current task (in progress): Add Stripe payment integration
Status: pending
Priority: High

I'll create a feature branch for this.

Suggested branch name: feature/stripe-payment-integration

Create this branch?"

User: "Yes"

Claude: "Branch created and checked out!

Branch: feature/stripe-payment-integration
Based on: dev
Task: Add Stripe payment integration

Ready to start working. When done, use /log then /push."
```

---

**Already on a feature branch:**

```
User: "/branch"

Claude: "You're currently on: feature/auth-system

This is already a feature branch. Would you like to:
1. Stay on this branch and continue working
2. Create a new branch for a different task
3. Switch to main/dev"

User: "Create new"

Claude: "Looking at pending tasks...

Next pending task: Dashboard layout
Priority: Medium

Suggested branch name: feature/dashboard-layout

Note: You have uncommitted changes on feature/auth-system.
Commit them first?"
```

---

**No task found:**

```
User: "/branch"

Claude: "I don't see any in-progress or pending tasks in tasks.md.

Would you like to:
1. Check your task list (/tasks)
2. Provide a branch name manually
3. Create a task first (/feature-planner)"

User: "user-settings"

Claude: "Creating branch with your name...

Branch: feature/user-settings
Based on: dev

Branch created!

Note: Consider adding this as a task in tasks.md to track progress."
```

---

**Custom branch name:**

```
User: "branch hotfix-production-error"

Claude: "Creating branch: fix/hotfix-production-error

Branch created and checked out!

Branch: fix/hotfix-production-error
Based on: dev

Ready to work on the hotfix."
```

## Error Handling

**Branch already exists:**
```
Branch 'feature/user-auth' already exists.

Would you like to:
1. Switch to existing branch
2. Create with different name (feature/user-auth-v2)
3. Delete and recreate (destructive!)
```

**Uncommitted changes blocking switch:**
```
You have uncommitted changes that would be overwritten:
- src/components/Auth.tsx
- lib/api.ts

Options:
1. Commit changes first (git commit)
2. Stash changes (git stash)
3. Cancel branch creation
```

**Not in a git repository:**
```
This directory is not a git repository.

Initialize git first:
git init
git remote add origin [your-repo-url]
```

## Integration with Other Skills

**With /tasks:**
Use task-tracker to find and start the next recommended task, then create its branch automatically.

**With /log:**
After finishing work, use /log to update changelog and mark task complete.

**With /push:**
Once branch is created and work is done, /push will push this branch to GitHub and create a PR.

**With /merge:**
After PR approval, /merge will merge the branch and clean it up.

## Branch Naming Best Practices

**Good names:**
- `feature/stripe-checkout`
- `fix/auth-redirect`
- `update/dashboard-ui`

**Bad names:**
- `feature/fix-stuff` (vague)
- `test` (no context)
- `manus-branch` (not descriptive)
- `feature/this-is-a-very-long-branch-name-that-goes-on-forever` (too long)
