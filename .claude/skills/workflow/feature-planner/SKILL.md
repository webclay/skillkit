---
name: feature-planner
description: Create detailed implementation plans for new features. Trigger words - plan feature, plan, design feature, scope, implementation plan, architect, spec out
---

# Feature Planner

Creates comprehensive implementation plans before you start building. Ensures you understand the full scope, get approval on UI components, and have a clear roadmap.

## When to Use This Skill

- User says "plan feature", "plan [feature name]", or "design feature"
- User wants to understand scope before implementing
- User mentions starting work on a new feature
- Before any significant new functionality

## Instructions

### Step 0: Create Feature Branch

1. Derive branch name from feature description:
   - "user settings page" -> `feature/user-settings`
   - "stripe integration" -> `feature/stripe-integration`
2. Run `git checkout -b feature/[name]`
3. Confirm branch created before proceeding

### Step 1: Understand the Feature

1. Read `project/projectbrief.md` for project context and existing patterns
2. Ask clarifying questions if needed:
   - What specific functionality is needed?
   - Who is this for (user type)?
   - Are there related features already built?
3. Search for existing code that might be relevant

### Step 2: Generate the Plan

Present the plan using this structure:

```
FEATURE PLAN: [Feature Name]
Branch: feature/[name]

## Overview
[One paragraph describing what this feature does and why]

## User Stories
- As a [user type], I want [action] so that [benefit]
- As a [user type], I want [action] so that [benefit]

## UI Mockup
[ASCII representation - see format below]

## Required Components

| Component | Status | Source |
|-----------|--------|--------|
| Button | Exists | @/components/ui/button |
| Dialog | Needs install | shadcn/ui (or your premium alternative) |
| DataTable | Needs install | shadcn/ui (or your premium alternative) |

**Action needed:** Which components should I install? You may have premium alternatives you'd prefer to use.

## Implementation Steps

1. [ ] Step one description
2. [ ] Step two description
3. [ ] Step three description
...

## Database Changes

| Table | Change | Fields |
|-------|--------|--------|
| users | Modify | Add `settings` JSON column |
| preferences | Create | id, userId, theme, notifications |

## API Endpoints

| Method | Route | Purpose |
|--------|-------|---------|
| GET | /api/settings | Fetch user settings |
| PATCH | /api/settings | Update user settings |

## Dependencies

| Package | Purpose | Approval |
|---------|---------|----------|
| zod | Validation | Pending |

**Action needed:** Approve packages before installation.

## Testing Strategy

- [ ] Unit tests for [component/function]
- [ ] Integration test for [flow]
- [ ] Manual test: [scenario]

## Risks & Considerations

- [Edge case or security consideration]
- [Performance consideration]
- [Dependency on other features]
```

### Step 3: Get Approval

After presenting the plan:

1. **Ask about components:** "Which components should I use? You may have premium alternatives."
2. **Ask about packages:** "Should I install these dependencies?"
3. **Confirm scope:** "Does this plan cover everything you need?"

Only proceed to implementation after explicit approval.

### Step 4: Create Task Entry

After approval, add the feature to `project/tasks.md`:

```markdown
## Task N: [Feature Name]
**Status:** pending
**Priority:** [High/Medium/Low]
**Tags:** #feature [relevant tags]
**Branch:** feature/[name]

### Checklist:
[Convert implementation steps to checkboxes]

### Implementation Notes:
[Link back to this plan discussion if needed]

### Test Strategy:
[Copy from plan]
```

## ASCII Mockup Format

Use this style for UI mockups:

```
+------------------------------------------+
|  Logo            Nav    Nav    [Avatar]  |
+------------------------------------------+
|          |                               |
| Sidebar  |     Main Content Area         |
| - Item   |                               |
| - Item   |   +---------------------+     |
| - Item   |   | Card Component      |     |
|          |   | - Content here      |     |
|          |   +---------------------+     |
|          |                               |
|          |   [Primary Button]            |
+------------------------------------------+
```

**Symbols:**
- `+--+` for borders/containers
- `|` for vertical separators
- `[ ]` for buttons
- `- Item` for list items
- Indent for nesting

## Component Handling Rules

**Critical:** Never install UI components without user consent.

1. **Check existing components first** - Scan `components/ui/` for what's already installed
2. **List all needed components** - Show which exist vs. need installation
3. **Mention premium alternatives** - User may have paid component libraries
4. **Wait for approval** - Do not run any install commands until confirmed

Example prompt:
```
These components need to be installed:
- Dialog (shadcn/ui default, or your premium alternative)
- DataTable (shadcn/ui default, or your premium alternative)

Which would you like me to install?
```

## Examples

**User:** "Plan feature: user settings page"

**Claude:**
1. Creates branch `feature/user-settings`
2. Reads project/projectbrief.md for context
3. Generates full plan with:
   - Overview of settings functionality
   - User stories for settings access
   - ASCII mockup of settings layout
   - Component list (existing + needed)
   - Implementation steps
   - Database changes for preferences
   - API endpoints
   - Testing strategy
4. Asks for component approval
5. After approval, creates task in project/tasks.md

---

**User:** "Plan the stripe integration"

**Claude:**
1. Creates branch `feature/stripe-integration`
2. Generates plan including:
   - Payment flow overview
   - User stories for checkout
   - ASCII mockup of payment UI
   - Components needed (pricing cards, checkout form)
   - Stripe SDK dependency (requires approval)
   - Webhook endpoints
   - Security considerations
3. Waits for approval before any installations
