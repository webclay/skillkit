---
name: project-setup
description: Initialize new projects or configure existing ones. Trigger words - setup, new project, initialize, init project, configure project, start new app, create app
---

# Project Setup

Sets up your project so Claude always knows what you're building, what tools you're using, and what to work on next. After setup, you can just describe what you want and Claude builds it consistently.

## When to Use This Skill

- User says "setup", "initialize", "get started", or "new project"
- project/projectbrief.md is empty or contains only template placeholders
- User describes a new app idea they want to build
- User asks "how do I start" or "help me begin"
- User wants to document an existing codebase

## Instructions

### Step 1: Detect Environment

Check what exists in the project:

1. **Check project/projectbrief.md** - Empty/template → questionnaire; Has content → analyze and recommend
2. **Check for existing code** - No code → fresh start; Has code → analyze codebase
3. **Check for installed skills** - Recommend missing skills based on project needs

### Step 2: Gather Information

**For new projects**, ask about (one question at a time):
- What you're building (in your own words)
- Who it's for
- Key features needed
- Any technology preferences (recommend if unsure)

**For existing codebases:**
- Scan package.json and file structure
- Detect frameworks, databases, auth systems
- Ask user to confirm or correct findings

See [questionnaire.md](questionnaire.md) for full question flow.
See [detection.md](detection.md) for codebase analysis patterns.

### The "No Assumptions" Principle

**Critical:** Never fill in projectbrief sections that the user didn't explicitly provide input for.

1. **Only document what the user said** - If they mentioned "user accounts and database", only fill those sections
2. **Use TBD markers** - For sections without user input, write: `[TBD - discuss before implementation]`
3. **Stop and ask when hitting TBD** - When starting a task that involves a TBD section, ask the user first
4. **Living document** - The projectbrief grows as you make decisions together, not all at once

**Example - User says:** "I want to build a habit tracker with user accounts"

**Fill in:**
- Project description: Habit tracker
- Features: User accounts

**Mark as TBD:**
- Database schema: `[TBD - discuss before implementation]`
- API endpoints: `[TBD - will define when building features]`
- UI framework: `[TBD - ask user preference before adding components]`
- Auth methods: `[TBD - email/password? social? discuss first]`

**Why this matters:** Users trust you to build what they want, not what you assume they want. Every technical decision should be either explicitly requested or discussed before implementation.

### Step 3: Create Documentation

Use templates from `.claude/templates/` to ensure consistency:

| File to Create | Template Source | Purpose |
|----------------|-----------------|---------|
| `.claude/project/projectbrief.md` | [projectbrief-template.md](../../../templates/projectbrief-template.md) | AI context - tech stack, features, patterns |
| `.claude/project/dev-context.md` | (created empty with structure) | Architectural decisions, patterns, security model |
| `.claude/project/tasks.md` | [tasks-template.md](../../../templates/tasks-template.md) | Task tracking with status icons |
| `.claude/project/changelog.md` | [changelog-template.md](../../../templates/changelog-template.md) | Session history log |
| `/docs/prd.md` (optional) | [prd-template.md](../../../templates/prd-template.md) | Human-readable requirements |

**dev-context.md initial content:**

```markdown
# Development Context

Project-specific architectural decisions and patterns.
This file is read by Claude before every task and persists across skill updates.

## When to Update This File

Add entries when:
- Making architectural decisions (security model, data access patterns)
- Establishing project-wide conventions (component structure, styling)
- Discovering project-specific quirks (linter setup, API patterns)
- Decisions that would confuse future AI sessions if not documented

Do NOT add:
- Temporary task status (use tasks.md)
- Session-specific notes (use changelog.md)
- Generic best practices (those belong in skills)

## Package Manager

[Will be populated during setup - see Step 5]

## Code Review

[Will be populated during setup if Greptile is selected - see questionnaire Q8]
```

**Process:**
1. Read the appropriate template file
2. Replace placeholders with user's answers from questionnaire
3. Remove sections that don't apply to this project (based on complexity)
4. Write the populated file to the target location

**Template Selection by Project Complexity:**
- **Simple projects** (landing page, small app): Use minimal sections from templates
- **Complex projects** (SaaS, multi-feature): Use full templates including schema, API endpoints, implementation priorities

See [wizard-flow.md](wizard-flow.md) for the complete setup flow.

**Important:** In projectbrief.md, include a "Relevant Skills" section that maps the tech stack to skills:

```markdown
## Relevant Skills

Based on your tech stack, reference these skills for implementation patterns:

| Technology | Skill Location |
|------------|----------------|
| Database (Prisma) | `.claude/skills/database/prisma/` |
| Auth (Better Auth) | `.claude/skills/auth/better-auth/` |
| UI (Shadcn) | `.claude/skills/ui/shadcn/` |
| Payments (Stripe) | `.claude/skills/payments/stripe/` |
```

This ensures Claude always knows which skills apply to this specific project.

### Step 4: Generate Environment Files

Create `.env` and `.env.production` files based on the user's confirmed tech stack, and ensure `.gitignore` is configured.

**Reference:** [env-vars-template.md](../../../templates/env-vars-template.md)

**Process:**

1. Read the env-vars-template
2. Determine the framework (TanStack Start, Next.js, etc.) from the user's confirmed stack
3. For each service the user selected, include that group's variables
4. Apply the framework prefix map - replace `[CLIENT]` variables with the correct prefix (e.g., `VITE_APP_URL` for TanStack Start)
5. Generate `.env` with localhost defaults pre-filled where applicable, API keys left empty
6. Generate `.env.production` with all values empty and helpful comments explaining where to find each value
7. Handle `.gitignore`:
   - If `.gitignore` exists: check if `.env` patterns are already present, append missing ones
   - If `.gitignore` doesn't exist: create it with env patterns plus common defaults (`node_modules/`, `.output/`, `dist/`, `.DS_Store`)

**Important rules:**
- Only include variable groups for services the user actually confirmed
- The App group (`APP_URL`) is always included
- Deduplicate: if `DATABASE_URL` appears in both database and auth groups, include it only once under the database section
- Railway's `RAILPACK_NO_SPA=1` goes in `.env.production` only (it's not needed locally)
- For TanStack Start: add a comment at the top of `.env`: `# Loaded automatically by Nitro (requires nitro.config.ts)`
- For Next.js: add a comment at the top: `# Next.js loads .env files automatically`
- No quotes around values in the generated files

**Tell the user:** "I've created your environment files. Fill in the API keys and secrets as you set up each service. The `.env.production` file has comments showing exactly where to find each value."

### Step 5: Detect Package Manager

Before recommending any commands, check for lock files:

| Lock File | Package Manager | Runner Command |
|-----------|-----------------|----------------|
| `bun.lockb` | bun | bunx |
| `pnpm-lock.yaml` | pnpm | pnpm dlx |
| `yarn.lock` | yarn | yarn dlx |
| `package-lock.json` | npm | npx |

If no lock file exists, **ask the user** which package manager they prefer. Default recommendation is **bun** (fastest).

**Important:** After determining the package manager, add it to `project/dev-context.md` so all skills and future sessions use the correct commands. Add a `## Package Manager` section:

```markdown
## Package Manager

This project uses **[detected/chosen package manager]**. Always use the matching commands:

| Action | Command |
|--------|---------|
| Install | `[pm] install` |
| Add package | `[pm] add [package]` |
| Run script | `[pm] run [script]` |
| Execute binary | `[runner] [binary]` |

Never use npm, npx, pnpm, or yarn unless they match the configured package manager above.
```

This goes in `project/dev-context.md` (not CLAUDE.md) because it's project-specific config that must survive SkillKit updates.

### Step 6: Recommend Skills

**Default Recommended Stack (for web apps):**

| Layer | Choice | Skill Location |
|-------|--------|----------------|
| Framework | TanStack Start | `.claude/skills/framework/tanstack-start/` |
| Database | Supabase | `.claude/skills/database/supabase/` |
| ORM | Drizzle | `.claude/skills/database/drizzle/` |
| Auth | Better Auth | `.claude/skills/auth/better-auth/` |
| UI | shadcn/ui | `.claude/skills/ui/shadcn/` |
| Payments | Stripe | `.claude/skills/payments/stripe/` |
| AI | TanStack AI | `.claude/skills/ai/tanstack-ai/` |

Based on project type, recommend relevant skills:

| Project Type | Recommended Skills |
|--------------|-------------------|
| Web app (SaaS) | tanstack-start, supabase, drizzle, better-auth, shadcn, stripe |
| Landing page | tanstack-start, shadcn |
| API/Backend | supabase, drizzle |
| Mobile app | expo, better-auth |
| AI application | tanstack-ai, supabase |
| Real-time features | supabase (use built-in Realtime) |

**Avoid recommending:**
- Next.js (use TanStack Start instead)
- Prisma for serverless (use Drizzle instead)
- Electric SQL with Supabase (use Supabase Realtime instead)

Explain why each is recommended and ask for approval.

### Step 7: Complete Setup

Final checks:
- All files created correctly
- Skills installed and configured
- Show user next steps

### Step 8: Mark Setup Complete

**Critical:** After all files are created successfully, create the setup marker:

1. Create `.claude/project/.setup-complete` file with content:
   ```
   Setup completed: [Current Date]
   Project: [Project Name]
   ```

2. This marker file tells Claude the project has been configured
3. Without this marker, Claude will prompt for setup on every conversation

**Why this matters:** The `.setup-complete` file is checked by CLAUDE.md on every conversation start. It's how Claude knows whether to proceed normally or trigger the setup wizard.

## Examples

**New Project Setup:**

```
User: "I want to build a task management app"

Claude: "Great! Let me set up your project.

Project name: task-manager (from your folder)
Package manager: bun (detected from bun.lockb)

Question 1: In 2-3 sentences, what does your app do?"

[... questionnaire continues one question at a time ...]

"Your project is now configured!

Project: Task Manager
Type: Web App
Package manager: bun

Recommended stack:
- TanStack Start (framework) - modern full-stack React
- Supabase (database) - managed Postgres with free tier
- Drizzle (ORM) - lightweight, great for serverless
- Better Auth (authentication) - flexible, works with any DB
- shadcn/ui (components) - copy-paste, customizable

To create the project:
bun create @tanstack/app task-manager

Next steps:
1. Review project/projectbrief.md
2. Start building - just tell me what you want!"
```

---

**Existing Codebase Setup:**

```
User: "Help me set up this project"

Claude: "I analyzed your codebase. Here's what I found:

Framework: TanStack Start
Database: Supabase (Postgres)
ORM: Drizzle
Auth: Better Auth
UI: Tailwind CSS + shadcn/ui
Package manager: bun

Is this correct? Let me know if I missed anything."
```
