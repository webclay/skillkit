# Project Instructions

## First-Time Setup Check

**On every conversation start, check if this project has been configured:**

1. Check if `.claude/project/.setup-complete` exists
2. If **NOT found**:
   - This project hasn't been set up with the skills system yet
   - Read `.claude/skills/workflow/project-setup/SKILL.md`
   - Check for existing code (`package.json`, `src/`, `app/`, etc.)
   - If code exists → Auto-detect stack and generate projectbrief
   - If no code → Run full questionnaire for new project
3. If **found**:
   - Project is configured, proceed normally
   - Read `project/projectbrief.md` before any task

---

Read `project/projectbrief.md` before any task.

## Core Files

All project-specific files are in `.claude/project/` (do not copy to other projects):

| File | Purpose |
|------|---------|
| `project/projectbrief.md` | Tech stack, features, design system |
| `project/tasks.md` | Current progress and task list |
| `project/changelog.md` | Session history |

Reusable across projects: `skills/`, `templates/`, `commands/`

## Communication Rules

**The user is a product engineer, not a programmer.** They describe what they want in plain language.

1. **Explain reasoning** - When presenting options, explain *why* one is better
2. **Always recommend** - Don't leave decisions to the user without guidance
3. **Summarize in plain language** - Describe outcomes, not code
4. **Ask about product, not tech** - Handle technical decisions yourself
5. **No code snippets** - Unless specifically requested

## Development Rules

1. **Pattern consistency** - Check existing code before building new features
2. **Ask before installing** - Never run `npm install` or `pnpm add` without approval
3. **Scope discipline** - Do only what was asked
4. **Run `npx ultracite fix`** - Before every commit

## Skills

**Before starting any task, check if a skill applies.** Skills in `.claude/skills/` provide patterns for specific technologies.

| When the task involves... | Use this skill |
|---------------------------|----------------|
| Database, schema, migrations, queries | `database/prisma`, `database/drizzle`, or `database/supabase` |
| User login, signup, sessions, OAuth | `auth/better-auth` |
| Payments, checkout, subscriptions | `payments/stripe` |
| UI components, forms, dialogs | `ui/shadcn` |
| AI features, chat, text generation | `ai/vercel-ai-sdk` |
| Sending emails, notifications | `email/resend` |
| Framework-specific patterns | `framework/nextjs` or `framework/tanstack-start` |
| Project setup, new projects | `workflow/project-setup` |
| Feature planning, "plan feature" | `workflow/feature-planner` |
| Saving progress, end of session | `workflow/session-log` |
| Task tracking, what's next | `workflow/task-tracker` |
| Server issues, port conflicts | `workflow/dev-server` |
| Creating new skills | `workflow/skill-creator` |
| Code review, quality checks | `workflow/code-reviewer` |
| Database security, access patterns | `security/database-security-audit` |
| Debugging errors | `workflow/debugger` |
| Linting before commits | `workflow/ultracite` |
| Development best practices | `workflow/best-practices` |
| Updating the skills system | `workflow/update-system` |
| Migrating from old claude-memory | `workflow/migrate-from-memory` |

## Common Mistakes to Avoid

### Dependencies
- Never install packages without asking first
- Check if project uses `npm` or `pnpm` before running install commands
- Don't add shadcn/UI components without user approval

### Scope
- Don't refactor unrelated code while fixing a bug
- Don't "improve" code that wasn't asked to be changed
- Don't add features that weren't requested
- Don't change file structure without discussion

### Patterns
- Match existing spacing and formatting
- Use existing component variants, don't create duplicates
- Check existing code before creating new patterns
- Follow the naming conventions already in the codebase
- Follow the design system in project/projectbrief.md

### Code Quality
- Remove console.log statements before committing
- Use proper TypeScript types, avoid `any`
- Run linter before suggesting commits
- Keep solutions simple - don't over-engineer
- Handle error cases

### Communication
- Ask clarifying questions instead of assuming
- Warn about side effects before making changes
- Get confirmation before large changes

### Formatting
- Never use em dashes, always use hyphens (-)
