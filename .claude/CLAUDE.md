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

## SkillKit Version Check

**On conversation start (after setup check), check for SkillKit updates:**

1. Read `.claude/version.json` to get local version
2. Fetch latest version from GitHub: `https://api.github.com/repos/grasman79/skillkit/releases/latest`
3. Compare versions (semantic versioning)
4. If update available, show notification:
   ```
   SkillKit update available: v1.1.0 → v1.2.0
   Run "skillkit update" to install.
   ```
5. If check fails (network error, etc.), skip silently - don't block the user

**Skip this check if:**
- User is in the middle of a task (not conversation start)
- Last check was less than 24 hours ago (check `.claude/project/.last-version-check`)

---

Read `project/projectbrief.md` and `project/dev-context.md` before any task.

## Core Files

All project-specific files are in `.claude/project/` (do not copy to other projects):

| File | Purpose |
|------|---------|
| `project/projectbrief.md` | Tech stack, features, design system |
| `project/dev-context.md` | Architectural decisions, patterns, security model |
| `project/tasks.md` | Current progress and task list |
| `project/changelog.md` | Session history |

Reusable across projects: `skills/`, `templates/`, `commands/`

**Important:** Never put project-specific information into skills. Skills are generic and reusable across all projects. When a skill update happens, skill files get overwritten. All project-specific decisions go in `project/dev-context.md`.

## Communication Rules

**The user is a product engineer, not a programmer.** They describe what they want in plain language.

1. **Explain reasoning** - When presenting options, explain *why* one is better
2. **Always recommend** - Don't leave decisions to the user without guidance
3. **Summarize in plain language** - Describe outcomes, not code
4. **Ask about product, not tech** - Handle technical decisions yourself
5. **No code snippets** - Unless specifically requested

## Development Rules

1. **Pattern consistency** - Check existing code before building new features
2. **Ask before installing** - Never add packages without approval
3. **Scope discipline** - Do only what was asked
4. **Run linter before every commit** - Use the project's configured package manager runner (check `## Package Manager` in `project/dev-context.md` or detect from lock files)

## Automated Git Workflow

**Use these skills to automate your feature branch workflow:**

### The Complete Workflow

1. **`/branch`** - Auto-create feature branch from current task
2. **[Work on code]** - Make your changes
3. **`/wrap-up`** - Log, lint, commit, push, create PR, handle code review, merge, clean up. Done.

### When to Use Each Command

| Command | When to use | What it does |
|---------|-------------|--------------|
| `/branch` | Starting a new task | Creates feature/task-name branch from tasks.md |
| `/wrap-up` | Done working | Logs session, lints, builds, commits, pushes, creates PR, waits for Greptile review (if configured) and auto-fixes issues, merges, cleans up branch |
| `/log` | Mid-session progress save | Updates changelog.md and tasks.md (no push) |

**Important:** Never push directly to dev/main. Always use feature branches with PRs.

## Skills

**Before starting any task, check if a skill applies.** Skills in `.claude/skills/` provide patterns for specific technologies.

| When the task involves... | Use this skill |
|---------------------------|----------------|
| Creating a feature branch | `workflow/create-branch` |
| End of session (log + commit + push + merge) | `workflow/wrap-up` |
| Saving session progress (no push) | `workflow/session-log` |
| Task tracking, what's next | `workflow/task-tracker` |
| Database, schema, migrations, queries | `database/prisma`, `database/drizzle`, or `database/supabase` |
| User login, signup, sessions, OAuth | `auth/better-auth` |
| Payments, checkout, subscriptions | `payments/stripe` |
| UI components, forms, dialogs | `ui/shadcn` |
| AI features, chat, text generation | `ai/vercel-ai-sdk` |
| Sending emails, notifications | `email/resend` |
| File uploads, image storage, S3/R2 | `storage/cloudflare-r2` |
| Framework-specific patterns | `framework/nextjs` or `framework/tanstack-start` |
| Deploying to Railway (TanStack Start SSR) | `deployment/railway` |
| Project setup, new projects | `workflow/project-setup` |
| Feature planning, "plan feature" | `workflow/feature-planner` |
| Server issues, port conflicts | `workflow/dev-server` |
| Creating new skills | `workflow/skill-creator` |
| Code review, quality checks | `workflow/code-reviewer` |
| Database security, access patterns | `security/database-security-audit` |
| Auth security, session management | `security/auth-security-audit` |
| API security, input validation | `security/api-security-audit` |
| Frontend security, XSS, dependencies | `security/frontend-security-audit` |
| Deployment security, headers, secrets | `security/deployment-security-audit` |
| TanStack Start security headers (Nitro) | `security/tanstack-nitro-security` |
| Debugging errors | `workflow/debugger` |
| Linting before commits | `workflow/ultracite` |
| Development best practices | `workflow/best-practices` |
| Updating SkillKit | `workflow/update-system` (trigger: "skillkit update") |
| Migrating from old claude-memory | `workflow/migrate-from-memory` |
| Updating dependencies | `workflow/update-stack` |
| Background jobs, workflows, event-driven | `backend/motia` |

## Common Mistakes to Avoid

### Dependencies
- Never install packages without asking first
- Detect package manager from lock files (`bun.lockb`, `pnpm-lock.yaml`, etc.) or ask user
- Default recommendation is **bun** (fastest)
- Don't add shadcn/UI components without user approval

### Scope
- Don't refactor unrelated code while fixing a bug
- Don't "improve" code that wasn't asked to be changed
- Don't add features that weren't requested
- Don't change file structure without discussion
- **UI/styling changes:** Only modify the specific elements requested. Do not over-apply themes, colors, or styles to unrelated components (e.g., if asked to change button colors, don't touch sidebar, badges, borders, or navigation)

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
- Keep solutions simple — don't over-engineer
- Handle error cases

### Communication
- Ask clarifying questions instead of assuming
- Warn about side effects before making changes
- Get confirmation before large changes

### Formatting
- Never use em dashes (—), always use hyphens (-)
