---
name: merge-branch
description: Merge an approved PR into dev/main and clean up branches. Trigger words - merge, merge branch, merge PR, complete PR
---

# Merge Branch

Merges an approved pull request on GitHub into the target branch (dev or main), then cleans up local and remote branches. Works with PRs created by /push.

## When to Use This Skill

- User says "merge", "merge branch", or "complete PR"
- Pull request has been approved by Grablite or reviewers
- User wants to complete a feature and clean up branches

## Workflow

### Step 1: Check Current Branch and PR Status

```bash
git branch --show-current
```

Note the current feature branch.

Check if a PR exists for this branch:

```bash
gh pr list --head feature/branch-name
```

**If no PR found:**
```
No pull request found for branch: feature/branch-name

Have you pushed this branch yet?
Use /push to create a PR first.
```

Stop here.

**If PR found:**
Get PR number and status.

### Step 2: Check PR Approval Status

```bash
gh pr view [PR_NUMBER] --json state,reviewDecision,mergeable
```

**Check conditions:**
- State: `OPEN`
- Review Decision: `APPROVED` (or no review required)
- Mergeable: `MERGEABLE`

**If not approved:**
```
PR #42 is not approved yet.

Status: Changes requested / Pending review

Would you like to:
1. Wait for approval
2. View PR to see feedback: gh pr view 42 --web
3. Merge anyway (not recommended)
```

**If conflicts exist:**
```
PR #42 has merge conflicts.

You need to resolve conflicts first:
1. git checkout dev
2. git pull origin dev
3. git checkout feature/branch-name
4. git merge dev
5. Resolve conflicts
6. git push
```

### Step 3: Merge PR on GitHub

Use GitHub CLI to merge the PR:

```bash
gh pr merge [PR_NUMBER] --squash --delete-branch
```

**Merge strategies:**
- `--squash` - Combine all commits into one (default, cleanest history)
- `--merge` - Preserve all commits (use for complex features)
- `--rebase` - Rebase and merge (linear history)

Use `--squash` by default unless user specifies otherwise.

**Flags:**
- `--delete-branch` - Automatically delete remote branch after merge
- `--auto` - Merge automatically when checks pass

### Step 4: Update Local Repository

Switch to target branch and pull the merge:

```bash
git checkout dev
git pull origin dev
```

### Step 5: Delete Local Feature Branch

```bash
git branch -d feature/branch-name
```

**If branch has unmerged commits (shouldn't happen):**
```bash
git branch -D feature/branch-name
```

Only use `-D` if user confirms.

### Step 6: Confirm Completion

```
PR Merged Successfully!

PR: #42
Merged: feature/stripe-integration -> dev
Strategy: Squash and merge

Branches cleaned up:
✓ Remote branch deleted
✓ Local branch deleted

You're now on: dev
Next: Use /branch to start a new feature
```

## Examples

**Merge approved PR:**

```
User: "/merge"

Claude: "Checking PR status...

Current branch: feature/stripe-integration
PR: #42
Status: Approved ✓
Mergeable: Yes ✓

Ready to merge PR #42 into dev?"

User: "Yes"

Claude: "Merging PR on GitHub...

PR Merged Successfully!

PR: #42
Merged: feature/stripe-integration -> dev
Strategy: Squash and merge

Branches cleaned up:
✓ Remote branch deleted
✓ Local branch deleted

You're now on: dev
Next: Use /branch to start a new feature"
```

---

**PR not approved yet:**

```
User: "/merge"

Claude: "Checking PR status...

PR #38 is not approved yet.

Status: Review requested from Grablite
Comments: 2 suggestions

Would you like to:
1. View PR feedback: gh pr view 38 --web
2. Wait for approval
3. Address feedback first"
```

---

**No PR exists:**

```
User: "/merge"

Claude: "No pull request found for branch: feature/dashboard-ui

Have you pushed this branch yet?

Next steps:
1. Use /push to create a PR
2. Wait for Grablite review
3. Then use /merge"
```

---

## Error Handling

**Merge conflicts on GitHub:**
```
PR #42 has merge conflicts that must be resolved.

To resolve:
1. Update your branch with latest dev:
   git checkout feature/branch-name
   git pull origin dev
2. Fix conflicts in your editor
3. Commit resolved changes:
   git add .
   git commit -m "Resolve merge conflicts"
4. Push:
   git push
5. Try /merge again
```

**No PR found:**
```
No pull request found for this branch.

Did you forget to /push?
```

**GitHub CLI not installed:**
```
GitHub CLI (gh) is not installed.

Install it to use automated PR merging:
- Mac: brew install gh
- Other: https://cli.github.com/

Or merge manually on GitHub, then:
1. git checkout dev
2. git pull origin dev
3. git branch -d feature/branch-name
```

**Not authenticated with GitHub:**
```
You're not authenticated with GitHub CLI.

Run: gh auth login

Then try /merge again.
```

## Merge Strategies

**Squash (default):**
- Combines all commits into one
- Cleanest history
- Good for features with many small commits
- Command: `gh pr merge --squash`

**Merge commit:**
- Preserves all individual commits
- Full history maintained
- Good for complex features with meaningful commit history
- Command: `gh pr merge --merge`

**Rebase:**
- Linear history
- No merge commit
- Good for simple changes
- Command: `gh pr merge --rebase`

By default, use **squash** unless user specifies otherwise.

## Integration with Other Skills

**After /merge:**
- Use `/branch` to start working on the next task
- Use `/tasks` to see what's next
- Your local repo is now on `dev` with the merged changes

**With /log:**
- /log should be used BEFORE /push (not before /merge)
- /merge happens after PR approval, so changelog is already updated

**Workflow reminder:**
1. `/branch` - Create feature branch
2. [Work on code]
3. `/log` - Update changelog and tasks
4. `/push` - Push and create PR
5. [Wait for Grablite review]
6. `/merge` - Merge approved PR and cleanup
