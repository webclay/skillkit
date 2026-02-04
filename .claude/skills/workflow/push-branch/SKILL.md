---
name: push-branch
description: Push current branch to GitHub and create a pull request. Trigger words - push, push branch, create PR, submit for review, ready for review
---

# Push Branch

Prepares your code for a pull request by running the linter, committing changes, pushing to GitHub, and creating a PR. Works with branches created by /branch or existing feature branches.

## When to Use This Skill

- User says "push", "push branch", or "ready for review"
- Code changes are ready to be submitted for review
- User wants to create a pull request

## Workflow

### Step 1: Check Current Branch

```bash
git branch --show-current
```

**If on main/dev/master:**
```
You're on the main branch. You need to create a feature branch first.

Would you like to:
1. Use /branch to create a branch from your current task
2. Create a branch manually with a custom name

(Pushing directly to main/dev is not recommended)
```

Stop here and wait for user to create a branch.

**If on a feature branch:**
Continue with the push workflow.

### Step 2: Run Linter

```bash
npx ultracite fix
```

If linter finds issues that can't be auto-fixed, show them and ask user to fix before continuing.

### Step 3: Check for Changes

```bash
git status
```

**If no changes:**
```
No changes detected. Nothing to push.

If you expected changes, check:
- Are files saved?
- Are changes in .gitignore?
```

Stop here.

**If changes exist:**
Continue to commit.

### Step 4: Generate Commit Message

Review the changes and generate a commit message:

```bash
git diff --staged
git diff
```

**Commit message format:**
```
[Type]: Brief description

- Detail 1
- Detail 2

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
```

**Types:**
- `Feature` - New functionality
- `Fix` - Bug fix
- `Update` - Enhancement to existing feature
- `Refactor` - Code restructuring
- `Chore` - Maintenance, deps, config

Suggest a commit message based on changes, or ask user if unclear.

### Step 5: Stage and Commit

```bash
git add .
git commit -m "$(cat <<'EOF'
[Generated commit message here]
EOF
)"
```

### Step 6: Push to GitHub

Check if branch exists on remote:

```bash
git ls-remote --heads origin feature/branch-name
```

**If new branch:**
```bash
git push -u origin feature/branch-name
```

**If existing branch:**
```bash
git push origin feature/branch-name
```

### Step 7: Create Pull Request

Use GitHub CLI to create the PR:

```bash
gh pr create --base dev --title "PR Title" --body "$(cat <<'EOF'
## Summary
[What changed and why]

## Changes
- Change 1
- Change 2

## Test Plan
- [ ] Tested feature X
- [ ] Verified Y works

## Notes
[Any important context]

🤖 Created with Claude Code
EOF
)"
```

**PR Title Format:**
- Keep under 70 characters
- Use same format as commit: `Feature: Add user authentication`

**If `gh` CLI not available:**
Provide the manual PR creation link instead.

Show success message with PR details:

**If PR created via `gh`:**
```
Pull Request Created!

Branch: feature/branch-name
PR: #123
URL: [PR URL from gh output]

Grablite will review your PR shortly (usually 3-5 minutes).

Next steps:
1. Wait for Grablite's review
2. Address any feedback if needed
3. Use /merge once approved
```

**If manual PR link provided:**
```
Branch pushed to GitHub!

Branch: feature/branch-name
Commits: 1 new commit

Create your Pull Request:
[Compare URL]

Next steps:
1. Click the link above to create PR
2. Add title and description
3. Wait for Grablite review (3-5 minutes)
4. Use /merge after approval
```

## Examples

**Push from feature branch:**

```
User: "/push"

Claude: "Preparing your code for review...

Current branch: feature/stripe-integration
Running linter... ✓ All checks passed

Changes detected:
- src/lib/stripe.ts
- app/checkout/page.tsx
- package.json

Suggested commit message:
Feature: Add Stripe checkout integration

- Implement checkout flow
- Add webhook handler
- Configure Stripe SDK

Use this message?"

User: "Yes"

Claude: "Committed and pushing...

Pull Request Created!

Branch: feature/stripe-integration
PR: #42
URL: https://github.com/username/project/pull/42

Grablite will review shortly (3-5 minutes).

Next steps:
1. Wait for review
2. Address feedback if needed
3. Use /merge once approved"
```

---

**Push from main branch (blocked):**

```
User: "/push"

Claude: "You're on the dev branch.

You need to create a feature branch first. Pushing directly to dev is not recommended.

Would you like to:
1. Use /branch to create a branch from your current task
2. Provide a custom branch name"

User: "/branch"

Claude: [Creates branch based on task, then user can /push]
```

---

## Error Handling

**No changes to commit:**
```
No changes detected. Nothing to push.

If you expected changes, check:
- Are files saved?
- Are changes in .gitignore?
```

**Push rejected (branch exists):**
```bash
git push --force-with-lease origin feature/branch-name
```
Only use if the user confirms they want to overwrite.

**Lint errors that can't be auto-fixed:**
```
Linter found issues that need manual fixes:

[error details]

Please fix these before pushing, or type "skip lint" to push anyway.
```
