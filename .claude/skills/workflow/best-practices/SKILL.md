---
name: best-practices
description: Development best practices for task management, commits, and session workflows. Trigger words - best practices, code quality, commit messages, task tracking, workflow, conventions
---

# Best Practices

Guidelines for effective development workflows including task management, commit messages, and session protocols.

## When to Use This Skill

- User asks about best practices for development
- Creating or reviewing commit messages
- Managing tasks and tracking progress
- Starting or ending a development session
- Discussing code quality standards

## Task Management

### Task Tagging System

Every task in `project/tasks.md` should include status and context:

```markdown
## Task 3: User Authentication
**Status:** in progress
**Priority:** High
**Tags:** #backend #auth #security
**Dependencies:** Task 1, Task 2
```

**Standard Tags:**
- **Domain:** `#frontend`, `#backend`, `#database`, `#api`, `#ui`, `#devops`
- **Type:** `#feature`, `#bug`, `#refactor`, `#docs`, `#test`, `#security`
- **Priority:** `#critical`, `#high`, `#medium`, `#low`

### Automatic Updates

Update `project/tasks.md` when:
- Creating a commit → Mark related tasks as in_progress or complete
- User says "done with [task]" → Update status to complete
- User says "starting [task]" → Update status to in_progress
- Build fails → Add note to relevant task with error details

## Commit Workflow

### Smart Commit Messages

When user says "commit", follow this process:

1. **Analyze changes** - What files were modified?
2. **Match to tasks** - Which task does this relate to?
3. **Update tasks** - Mark checkboxes complete
4. **Generate message** - Use conventional commit format

### Commit Message Format

```
<type>(<scope>): <short description>

<detailed description>

Changes:
- [Bullet point of changes]
- [Another change]

Related: Task X
```

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `refactor` - Code restructuring
- `docs` - Documentation
- `test` - Tests
- `chore` - Maintenance

**Example:**
```
feat(auth): implement login form with validation

Add login form component with email/password fields.
Integrate form validation with Zod.
Add error handling and loading states.

Changes:
- Add LoginForm component
- Add validation schema
- Add auth API route

Related: Task 3
```

## Session Protocols

### Session Start

When starting a new session:

1. **Read** `project/tasks.md` to understand current state
2. **Check** `dev-context.md` for in-progress work
3. **Identify** last worked-on task
4. **Ask user** what they want to work on

**Example greeting:**
```
I see you were working on Task 3 (User Authentication).
Last session you completed the login form.

Would you like to continue with the signup flow, or work on something else?
```

### Session End

When user indicates session is ending:

1. **Review** uncommitted changes
2. **Prompt** to commit if work was done
3. **Update** `project/tasks.md` with progress
4. **Update** `dev-context.md` with current state
5. **Suggest** next steps

**Example wrap-up:**
```
Before you go, let me wrap up:

Uncommitted changes:
- Modified: auth.ts, login.tsx
- Added: validation.ts

Would you like to commit these changes?

I'll update tasks.md with today's progress:
- Completed: Login form component
- In progress: Form validation

Next session suggestion: Add error handling and loading states.
```

## Code Quality Gates

### Pre-Commit Checks

Before allowing commit, verify:

- [ ] No console.log statements (unless approved)
- [ ] No TODO comments without issue reference
- [ ] TypeScript strict mode compliance
- [ ] Related task checkboxes updated
- [ ] Run `npx ultracite fix` for linting

### Quality Standards

- **TypeScript** - Use proper types, avoid `any`
- **Error handling** - Handle error cases appropriately
- **Naming** - Follow existing conventions in codebase
- **Simplicity** - Don't over-engineer solutions

## Pattern Detection

### When to Document Patterns

Suggest creating documentation when:
- Same code pattern used 3+ times
- New technology added to project
- Performance optimization discovered
- Security fix applied

**Example suggestion:**
```
I notice you're using this API pattern repeatedly.
Should I document it in the relevant skill file for consistency?
```

## Progress Tracking

### Velocity Awareness

Track informally:
- Tasks completed per session
- Blockers encountered
- Estimation accuracy

### Lessons Learned

After completing complex tasks, note:
- What worked well
- What took longer than expected
- Patterns to reuse

## Quick Commands

Users can say:
- "status" → Show current task status
- "next" → Suggest next task
- "commit" → Create commit with proper message
- "wrap up" → End session protocol
- "blockers" → List blocked tasks

## Key Principles

1. **Proactive updates** - Update tasks as you work, not just at the end
2. **Clear commits** - Each commit should be self-documenting
3. **Context preservation** - Always update dev-context for in-progress work
4. **Pattern consistency** - Check existing code before creating new patterns
5. **Simplicity** - Don't over-engineer, keep solutions focused
