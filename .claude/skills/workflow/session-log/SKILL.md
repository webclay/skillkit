---
name: session-log
description: Save session progress to changelog and update task status. Trigger words - save progress, log session, done for now, end session, wrap up, save work, changelog
---

# Session Log

Saves what you accomplished in this session so future Claude sessions know what was already done, your task list stays up to date, and you have a history of your project's progress.

## When to Use This Skill

- User says "save progress", "log this session", or "we're done for now"
- Session is ending and work was accomplished
- User asks to update the changelog or mark tasks complete
- User wants to record what was built today
- Significant milestone reached that should be documented

## Instructions

### Step 1: Gather What Was Done

Review the conversation to identify:
- Features built or modified
- Bugs fixed
- Tasks completed
- Key decisions made
- Files changed

Also check:
- Git commits made (if any): `git log --oneline -5`
- Current state of tasks in project/tasks.md

### Step 1.5: Capture Dev Context

For any feature still in progress, create or update `.claude/dev-context.md`:

**What to capture:**
- Current state of the feature (what works, what doesn't)
- Key decisions made and why (especially non-obvious ones)
- Critical files to reference next session
- Exact next step to continue
- Any gotchas or blockers discovered

**Format:**

```markdown
## [Feature Name] - Last updated: [Date]

### State
[Brief status - what's working, what's left]

### Decisions
- [Decision]: [Reason]

### Key Files
- [file]: [what it does]

### Continue From
[Exact next action to take]

### Watch Out For
- [Gotcha or blocker]
```

**When to update:**
- Feature is in progress and session is ending
- Major decision was made that affects future work
- Discovered something non-obvious about the codebase

**When to skip:**
- All features were completed this session
- Only quick fixes or minor changes were made

### Step 2: Update Tasks File

In `.claude/project/tasks.md`:
- Mark completed tasks as done (change status, check all boxes)
- Update partially finished tasks (check completed boxes, update status)
- Add implementation notes if relevant decisions were made
- Update "Last Updated" timestamp

### Step 3: Update Changelog

In `.claude/project/changelog.md`, add new entry at top of "Session Log" section:

```markdown
## [Date] - [Brief Session Title]

### Completed
- [Feature/task 1]
- [Feature/task 2]

### Key Decisions
- [Important decision and why]

### Files Changed
- [file1.tsx]
- [file2.ts]

### Next Steps
- [What to work on next]
```

### Step 4: Confirm to User

Show summary:

```
Session Logged!

Date: [Today's Date]
Title: [Session Focus]

Completed:
- [Feature 1]
- [Feature 2]

Tasks Updated:
- [Task name] → completed
- [Task name] → in progress

Your progress is saved for next time.
```

## Examples

**End of Session:**

```
User: "Save my progress"

Claude: "Session Logged!

Date: January 4, 2026
Title: User Authentication Setup

Completed:
- Login form with validation
- Google OAuth integration
- Protected route middleware

Tasks Updated:
- User Authentication → completed
- Dashboard Layout → in progress

Files Changed:
- app/sign-in/page.tsx
- lib/auth.ts
- middleware.ts

Your progress is saved for next time."
```

---

**After Completing a Feature:**

```
User: "Log what we built"

Claude: "Session Logged!

Date: January 4, 2026
Title: Payment Integration

Completed:
- Stripe checkout integration
- Webhook handler for payment events
- Subscription management UI

Key Decisions:
- Using Stripe Checkout (hosted) instead of custom form
- 7-day free trial for all paid plans

Next Steps:
- Add invoice history page
- Set up email receipts"
```

---

**With Feature In Progress:**

```
User: "Save my progress"

Claude: "Session Logged!

Date: January 4, 2026
Title: Dashboard Implementation

Completed:
- Dashboard layout with sidebar
- Stats cards component

In Progress:
- Activity feed (captured in dev-context.md)

Dev Context Updated:
- Activity Feed feature context saved
- Key files and next steps documented

Your progress is saved for next time."
```

**dev-context.md created:**
```markdown
## Activity Feed - Last updated: January 4, 2026

### State
Layout complete, data fetching works. Still need: pagination, real-time updates, empty state.

### Decisions
- Using Server-Sent Events for real-time (simpler than WebSockets for our use case)
- Infinite scroll instead of numbered pages

### Key Files
- app/dashboard/activity/page.tsx - Main feed component
- lib/activity.ts - Data fetching and SSE setup
- components/activity-item.tsx - Individual activity card

### Continue From
Implement pagination - the `loadMore` function is stubbed but needs API integration

### Watch Out For
- SSE connection needs cleanup on unmount (memory leak otherwise)
- Rate limiting on activity API is 100 req/min
```
