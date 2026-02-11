---
name: wrap-up
description: End-of-session command that logs progress, lints, commits, pushes, creates a PR with auto-merge, and cleans up branches. Trigger words - wrap up, wrap-up, done for today, ship it, end session and push
---

# Wrap Up

Single command to close out a work session. Handles everything: logs progress, lints, commits, pushes, creates a PR with auto-merge, and cleans up your local branch. You walk away and it's done.

## When to Use This Skill

- User says "wrap up", "wrap-up", "done for today", or "ship it"
- Session is ending and user wants to save, commit, and push everything
- User wants the full end-of-session ritual in one command

## What This Does

1. Log the session (changelog + tasks)
2. Run linter and fix issues
3. Verify build passes
4. Commit all changes
5. Push to remote
6. Create PR (feature branches)
7. Wait for code review and auto-fix if needed (optional - Greptile)
8. Merge and clean up

## Workflow

### Step 1: Detect Package Manager

Check `project/dev-context.md` for a `## Package Manager` section to determine the runner command:

| Package Manager | Runner |
|-----------------|--------|
| bun | bunx |
| pnpm | pnpm dlx |
| npm | npx |
| yarn | yarn dlx |

If not specified, check for lock files (`bun.lockb`, `pnpm-lock.yaml`, etc.). Default to `bunx` if nothing found.

### Step 2: Log the Session

Review the conversation and identify:
- Features built or modified
- Bugs fixed
- Tasks completed
- Key decisions made

**Update `.claude/project/changelog.md`** - Add entry at the top:

```markdown
## [YYYY-MM-DD] - [Brief Session Title]

### Completed
- [What was done]

### Key Decisions
- [Important decisions, if any]

### Next Steps
- [What to work on next]
```

**Update `.claude/project/tasks.md`** - Mark completed tasks as done, update in-progress items.

**Update dev-context.md** (if features are still in progress) - Capture state for next session.

### Step 3: Update README Date

If a `README.md` exists in the project root with a date field, update it to today's date.

### Step 4: Run Linter

```bash
[runner] ultracite fix
```

If there are errors that can't be auto-fixed, fix them manually before continuing. Do not skip or bypass the linter.

### Step 5: Verify Build (if applicable)

If the project has a build command in package.json:

```bash
[package-manager] run build
```

If build fails, fix the errors before committing. Do not push broken code.

### Step 6: Stage and Commit

```bash
git add .
git status
```

Review what's being committed. Then generate a commit message based on all changes:

```bash
git commit -m "$(cat <<'EOF'
[Type]: Brief description

- Detail 1
- Detail 2

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

**Commit types:** Feature, Fix, Update, Refactor, Chore

### Step 7: Push to Remote

```bash
git push origin [current-branch]
```

If the branch doesn't exist on remote yet:

```bash
git push -u origin [current-branch]
```

### Step 8: Create PR (feature branches only)

**If on a feature branch** (not main/dev/master):

Create the PR:

```bash
gh pr create --base dev --title "[Type]: Brief description" --body "$(cat <<'EOF'
## Summary
[What changed and why]

## Changes
- Change 1
- Change 2

Created with Claude Code
EOF
)"
```

Note the PR number from the output.

**If on main/dev:** Skip PR creation, just push directly. Skip to Step 10.

### Step 9: Code Review Cycle (optional - Greptile users only)

**Check if Greptile is configured:** Look for a `## Code Review` section in `project/dev-context.md`. If it exists and specifies Greptile, run the review cycle. If not, skip to Step 9b.

#### Greptile File Limit Check

Before waiting for Greptile, check how many files are in the PR:

```bash
gh pr diff {pr_number} --name-only | wc -l
```

**If file count exceeds 100:**

Greptile has a 100-file limit per review. Show a warning:

```
Warning: This PR changes [X] files, which exceeds Greptile's 100-file limit.
Greptile will skip this review.

Options:
1. Merge without Greptile review (auto-merge)
2. Cancel and split into smaller PRs
```

If user chooses option 1 (or no response), fall back to auto-merge (Step 9b). If option 2, stop and let the user reorganize.

**If file count is 100 or fewer:** Proceed with Greptile review cycle below.

#### Step 9a: Greptile Review Cycle

**Poll for review:**

Wait for Greptile to post its review. Check every 30 seconds, up to 10 minutes:

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/reviews
```

Look for a review from Greptile (check the `user.login` field for greptile or greptileai).

**If no Greptile review appears after 10 minutes:**
Fall back to auto-merge (Step 9b). Show message: "Greptile review not detected - enabling auto-merge."

**If Greptile review appears:**

Read the review body to find the confidence score (Greptile provides a score like 3/5, 4/5, or 5/5).

Also read all review comments:

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments
```

**If confidence score >= threshold (default 4/5):**

```
Greptile review: [score]/5
No issues to fix. Merging...
```

Merge the PR:

```bash
gh pr merge [pr_number] --squash --delete-branch
```

Proceed to Step 10.

**If confidence score < threshold:**

```
Greptile review: [score]/5
[X] issues found. Fixing...
```

1. Read each Greptile comment and understand the issue
2. Fix the code based on the feedback
3. Run linter again: `[runner] ultracite fix`
4. Verify build passes (if applicable)
5. Commit the fixes:
   ```bash
   git add .
   git commit -m "$(cat <<'EOF'
   Fix: Address code review feedback

   - [Fix 1]
   - [Fix 2]

   Co-Authored-By: Claude <noreply@anthropic.com>
   EOF
   )"
   ```
6. Push: `git push origin [branch]`
7. Go back to the top of Step 9a and poll for a new review

**Maximum review cycles:** 3 attempts. If still below threshold after 3 fix cycles, merge anyway and show a warning:

```
Merged after 3 fix cycles. Greptile score: [score]/5
Some review comments may need manual attention.
```

#### Step 9b: No Code Review (auto-merge)

If no Greptile is configured, enable auto-merge:

```bash
gh pr merge --auto --squash --delete-branch
```

### Step 10: Clean Up Local Branch

**If on a feature branch:**

Switch back to dev and delete the local feature branch:

```bash
git checkout dev
git pull origin dev
git branch -d [feature-branch-name]
```

**If on main/dev:** Skip this step.

### Step 11: Confirm Everything

**With Greptile review:**

```
Session wrapped up!

Logged:
- Changelog updated (YYYY-MM-DD)
- Tasks updated: X completed, Y in progress

Code:
- Linter: passed
- Build: passed (or N/A)
- Committed: [commit message summary]
- Pushed to: [branch-name]
- PR: #XX (URL)
- Greptile review: [score]/5 - merged
- Local branch cleaned up, now on dev

Next session: [suggested next task from tasks.md]
```

**Without Greptile (auto-merge):**

```
Session wrapped up!

Logged:
- Changelog updated (YYYY-MM-DD)
- Tasks updated: X completed, Y in progress

Code:
- Linter: passed
- Build: passed (or N/A)
- Committed: [commit message summary]
- Pushed to: [branch-name]
- PR: #XX (URL) - auto-merge enabled
- Local branch cleaned up, now on dev

Next session: [suggested next task from tasks.md]
```

## Error Handling

**Linter fails and can't auto-fix:**
Fix the issues before continuing. Show what needs fixing and resolve it.

**Build fails:**
Fix build errors before committing. Do not push broken code.

**Push rejected:**
```bash
git pull --rebase origin [branch]
git push origin [branch]
```

If conflicts exist, resolve them and retry.

**No changes to commit:**
```
No code changes to commit, but session was logged.

Changelog and tasks have been updated.
```

Skip the commit/push/PR steps.

**Auto-merge not available on the repo:**
If `gh pr merge --auto` fails (auto-merge not enabled in repo settings), show a message:

```
Auto-merge is not enabled for this repository.

To enable it: GitHub repo Settings > General > Allow auto-merge

For now, merge manually from the PR page:
[PR URL]
```

**gh CLI not available:**
Push the branch and provide the manual PR creation URL instead.

## Examples

**Wrap-up with Greptile review (score passes):**

```
User: "/wrap-up"

Claude: "Wrapping up your session...

Logged:
- Changelog updated (2026-02-11)
- Tasks: 'Add contact search' marked complete

Code:
- Linter: passed (2 auto-fixes applied)
- Build: passed
- Committed: Feature: Add contact search with filters
- Pushed to: feature/contact-search
- PR: #47 created

Waiting for Greptile review...
Greptile review: 5/5 - no issues found
PR #47 merged and branches cleaned up
Now on: dev

Next session: Add bulk actions to contact list"
```

---

**Wrap-up with Greptile review (needs fixes):**

```
User: "/wrap-up"

Claude: "Wrapping up your session...

[... log, lint, build, commit, push, PR created ...]

Waiting for Greptile review...
Greptile review: 2/5 - 3 issues found

Fixing:
1. Missing null check in search handler
2. Unused import in results component
3. SQL injection risk in filter query

Fixes applied, re-pushing...
Waiting for re-review...
Greptile review: 5/5 - all issues resolved

PR #47 merged and branches cleaned up
Now on: dev

Next session: Add bulk actions to contact list"
```

---

**Wrap-up without Greptile (auto-merge):**

```
User: "/wrap-up"

Claude: "Wrapping up your session...

Logged:
- Changelog updated (2026-02-11)
- Tasks: 'Add contact search' marked complete

Code:
- Linter: passed
- Build: passed
- Committed: Feature: Add contact search with filters
- Pushed to: feature/contact-search
- PR: #47 - auto-merge enabled
- Switched to dev, local branch deleted

Next session: Add bulk actions to contact list"
```

---

**Wrap-up from dev branch (no PR):**

```
User: "wrap up"

Claude: "Wrapping up your session...

Logged:
- Changelog updated (2026-02-11)
- Tasks: 'Fix sidebar collapse bug' marked complete

Code:
- Linter: passed
- Committed: Fix: Sidebar collapse state persistence
- Pushed to: dev

Next session: Implement deal pipeline view"
```

---

**Wrap-up with no code changes:**

```
User: "wrap up"

Claude: "Wrapping up your session...

Logged:
- Changelog updated (2026-02-11)
- Tasks: planning notes updated

No code changes to commit.

Next session: Start building the Products module"
```
