---
name: task-tracker
description: Track project tasks, show progress, suggest next steps, and mark work complete. Trigger words - status, progress, what's next, tasks, todo, task done, mark complete, show tasks
---

# Task Tracker

Keeps track of what you're building - what's done, what you're working on, what's coming next, and what's blocked.

## When to Use This Skill

- User asks about project status or progress
- User asks "what should I work on" or "what's next"
- User says a task is done or finished
- User asks about blocked tasks or dependencies
- User wants to see their task list

## Instructions

### Show Status

**Triggers:** "status", "progress", "how am I doing", "where are we"

1. Read `project/tasks.md` to find current task and progress
2. Calculate completion percentage from checkboxes
3. Run `git status` for uncommitted changes
4. Show summary:

```
PROJECT STATUS

Current Task: User Authentication
Progress: 60% (3/5 items complete)

Completed:
- [x] Login form UI
- [x] Password validation
- [x] Database schema

Remaining:
- [ ] OAuth integration
- [ ] Session management

Next up: Dashboard Layout (after this task)
```

### Suggest Next Task

**Triggers:** "what's next", "what should I do", "suggest a task"

1. Read `project/tasks.md` for pending tasks
2. Check `project/projectbrief.md` for feature dependencies
3. Find tasks with all dependencies completed
4. Sort by priority (high → medium → low)
5. Recommend with reasoning:

```
NEXT TASK RECOMMENDATION

Recommended: Dashboard Layout

Why this task?
- All dependencies complete (Auth is done)
- High priority
- Natural follow-up to authentication work

Alternatives:
1. User Settings - medium priority, small scope
2. Email Notifications - medium priority, can do in parallel

Would you like to start "Dashboard Layout"?
```

### Mark Task Complete

**Triggers:** "done", "finished", "completed", "I finished [task]"

1. Find task in `project/tasks.md` (use task name if provided)
2. Update status to "complete" and check all boxes
3. Check what tasks are now unblocked
4. Suggest next task:

```
TASK COMPLETED!

Marked complete: User Authentication

Now unblocked:
- Dashboard Layout (was waiting on auth)
- User Settings (was waiting on auth)

Suggested next: Dashboard Layout
- High priority
- Natural continuation

Start working on Dashboard Layout?
```

### Show Blocked Tasks

**Triggers:** "blocked", "what's blocking", "dependencies"

1. Find tasks with status "blocked" or unmet dependencies
2. Show what's blocking each and blocker progress
3. Recommend which blocker to prioritize:

```
BLOCKED TASKS

3 tasks waiting on dependencies:

1. Payment Integration
   Blocked by: User Settings (40% complete)
   Needs: User profile with billing info

2. Email Notifications
   Blocked by: Dashboard Layout (not started)
   Needs: Notification preferences UI

Recommendation:
Focus on User Settings to unblock Payment Integration

Available now: 4 tasks (ask "what's next" to see them)
```

## Examples

**Checking Progress:**
```
User: "How's the project going?"

Claude: "PROJECT STATUS

Current Task: User Authentication
Progress: 60% (3/5 items complete)
..."
```

---

**Completing a Task:**
```
User: "I finished the login page"

Claude: "TASK COMPLETED!

Marked complete: User Authentication

Now unblocked:
- Dashboard Layout
- User Settings

Suggested next: Dashboard Layout"
```

---

**Finding Next Work:**
```
User: "What should I work on?"

Claude: "NEXT TASK RECOMMENDATION

Recommended: Dashboard Layout

Why? All dependencies complete, high priority..."
```
