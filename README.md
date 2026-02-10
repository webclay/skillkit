# SkillKit

> A skills system for Claude Code that teaches it exactly how to build your app, so you never have to explain the same thing twice.

**Version:** 1.1.0
**Author:** Manuel Merz
**License:** MIT
**Date:** 10/02/2026

---

## What is this?

This is a **skills system for Claude Code**. Copy one folder into your project, and Claude instantly knows:

- **What you're building** - Your app idea, features, and who it's for
- **How to build it** - The right patterns for your tech stack (TanStack Start, Supabase, Drizzle, etc.)
- **What you're working on** - Current tasks, progress, and where you left off
- **Security best practices** - Server-side authentication, protected routes, and secure patterns

**The core idea:** Skills auto-trigger based on what you're doing. Ask to "add payments" and Claude automatically uses the Stripe skill. Ask to "save my progress" and Claude updates your changelog and task list.

### Before & After

| Without Skills | With Skills |
|----------------|-------------|
| Re-explain your app every session | Describe once, Claude remembers forever |
| Claude forgets your tech stack | Claude knows exactly what tools you're using |
| Inconsistent code patterns | Same patterns applied consistently |
| "Where was I?" every morning | Progress tracked, context preserved |

---

## Quick Start

### New Project

1. **Copy the `.claude` folder** into your project root
2. **Start Claude Code** and say: *"Help me set up my project"*
3. **Answer a few questions** about what you're building
4. **Start building** - just describe what you want

### Existing Project

1. **Copy the `.claude` folder** into your project root
2. **Start Claude Code** and say: *"Help me set up my project"*
3. **Claude scans your codebase**, detects your tech stack, and generates documentation
4. **Confirm the findings** and start building

That's it. No configuration files to edit. No commands to memorize.

---

## How It Works

### What is a Skill?

A **skill** is a knowledge file that teaches Claude how to do something specific. Think of it like a reference guide that Claude reads when needed.

For example:
- The `stripe` skill contains patterns for payment integration
- The `better-auth` skill knows how to set up secure authentication
- The `shadcn` skill understands UI component patterns

**You don't need to read or edit these files.** Claude automatically uses the right skill based on what you're asking for. Just describe what you want in plain language.

### Skills Auto-Trigger

Skills are knowledge files that Claude reads when relevant. You don't invoke them - they activate automatically based on **trigger words** in what you say:

| What You Say | Skill Used | What Happens |
|--------------|------------|--------------|
| "Add user authentication" | `better-auth` | Sets up login, signup, sessions with your framework |
| "Create a database schema" | `prisma` or `drizzle` | Follows your ORM's patterns correctly |
| "Add payment processing" | `stripe` | Implements checkout, webhooks, subscriptions |
| "Add AI chat to my app" | `vercel-ai-sdk` | Sets up streaming, models, UI components |
| "Plan feature: user settings" | `feature-planner` | Creates detailed plan with ASCII mockups |
| "Save my progress" | `session-log` | Updates changelog, tasks, dev context |
| "What should I work on?" | `task-tracker` | Shows status, suggests next task |

**How trigger words work:** Each skill has specific words or phrases that activate it. For example, the `stripe` skill triggers on words like "payment", "checkout", "subscription", "billing". The `debugger` skill triggers on "error", "bug", "not working", "broken". You don't need to memorize these - just describe what you want naturally.

### Folder Structure

```
.claude/
├── CLAUDE.md              # Instructions Claude follows
├── README.md              # This file
├── project/               # Project-specific (don't copy to other projects)
│   ├── projectbrief.md    # Your app description
│   ├── tasks.md           # Task list and progress
│   ├── changelog.md       # Session history
│   └── .setup-complete    # Setup marker
├── skills/                # Reusable - copy to other projects
├── templates/             # Reusable - copy to other projects
├── commands/              # Reusable - copy to other projects
├── settings.json
└── version.json
```

### Core Files

These files give Claude context about your project:

| File | What It Does | When It's Used |
|------|--------------|----------------|
| `project/projectbrief.md` | Describes your app, tech stack, and features | Every conversation - Claude reads this first |
| `project/tasks.md` | Tracks what needs building and progress | When asking about status or next steps |
| `project/changelog.md` | Records session history | When saving progress or resuming work |
| `dev-context.md` | Preserves in-progress feature state | When picking up unfinished work |

### Sharing Skills Across Projects

The folder structure separates project-specific files from reusable ones:

**To share skills with a new project, copy these folders:**
- `skills/`
- `templates/`
- `commands/`

**Don't copy:**
- `project/` (unique to each codebase)

---

## Workflow

### Starting Your Day

Just start chatting. Claude reads your project files and knows:
- What you're building
- Your tech stack
- Current tasks and their status
- Where you left off last session

### During Development

Describe what you want in plain language:

```
"Add a user profile page with avatar upload"
"Fix the bug where users can't log out"
"Make the dashboard load faster"
```

Claude uses the right skills automatically - Prisma patterns for database work, Shadcn patterns for UI, etc.

### Planning Features

Say "plan feature: [name]" to get a comprehensive implementation plan:

```
"Plan feature: user settings page"
```

Claude will:
1. Create a git branch automatically
2. Generate a detailed plan with ASCII UI mockups
3. List all required components (asks before installing)
4. Show database changes, API endpoints, testing strategy
5. Wait for your approval before implementing

### Ending Your Session

Say "save my progress" and Claude:
1. Updates `project/changelog.md` with what was accomplished
2. Updates `project/tasks.md` with completed and remaining items
3. Captures context in `dev-context.md` for unfinished features

Next session, Claude picks up exactly where you left off.

---

## Available Skills

### Workflow
| Skill | What It Does |
|-------|--------------|
| `project-setup` | Initialize new projects with recommended stack (TanStack Start, Supabase, Drizzle) |
| `feature-planner` | Create detailed implementation plans with ASCII mockups |
| `session-log` | Save progress at session end |
| `task-tracker` | Track tasks and progress |
| `dev-server` | Manage dev servers, kill ports |
| `skill-creator` | Create new skills from successful patterns |
| `code-reviewer` | Review code for quality, security, maintainability |
| `debugger` | Systematically investigate and fix errors |
| `ultracite` | Run linting and TypeScript checks before commits |
| `update-stack` | Safely update all dependencies with breaking change detection |

### Framework
| Skill | What It Does |
|-------|--------------|
| `astro` | Content-focused sites with islands architecture |
| `nextjs` | Next.js App Router patterns |
| `tanstack-start` | TanStack Start full-stack patterns |

### Database
| Skill | What It Does |
|-------|--------------|
| `prisma` | Prisma ORM patterns |
| `drizzle` | Drizzle ORM patterns (recommended for serverless) |
| `supabase` | Supabase with Realtime subscriptions, RLS, and ORM integration |
| `convex` | Convex real-time backend |
| `electric-sql` | Electric SQL local-first sync (for self-hosted Postgres only) |
| `neon` | Neon serverless Postgres |

### Auth
| Skill | What It Does |
|-------|--------------|
| `better-auth` | Better Auth with server-side protection, tRPC integration, TanStack Start patterns |

### Payments
| Skill | What It Does |
|-------|--------------|
| `stripe` | Stripe checkout, webhooks, subscriptions |
| `polar` | Polar open-source monetization |

### UI
| Skill | What It Does |
|-------|--------------|
| `shadcn` | Shadcn UI component patterns |
| `untitledui` | Untitled UI React components |

### AI
| Skill | What It Does |
|-------|--------------|
| `vercel-ai-sdk` | Vercel AI SDK streaming and tools |
| `openrouter` | OpenRouter multi-model API |

### API
| Skill | What It Does |
|-------|--------------|
| `server-actions` | Next.js Server Actions |
| `orpc` | oRPC type-safe APIs |
| `unkey` | Unkey API key management |

### Backend
| Skill | What It Does |
|-------|--------------|
| `durable-streams` | Resumable AI streaming |
| `motia` | Event-driven workflows, background jobs, multi-step processing |
| `workflow-devkit` | Vercel durable workflows |

### Deployment
| Skill | What It Does |
|-------|--------------|
| `vercel` | Vercel deployment |
| `netlify-edge` | Netlify Edge Functions |
| `railway` | Railway deployment for TanStack Start with Nitro SSR |
| `appwrite` | Appwrite BaaS |

### Email
| Skill | What It Does |
|-------|--------------|
| `resend` | Resend transactional email |

### Platform
| Skill | What It Does |
|-------|--------------|
| `chrome-extension` | Chrome Extension (MV3) |
| `expo` | Expo mobile apps |
| `react-native` | React Native patterns |
| `wordpress` | WordPress plugins |

### Scraping
| Skill | What It Does |
|-------|--------------|
| `firecrawl` | Firecrawl web scraping |

### Tooling
| Skill | What It Does |
|-------|--------------|
| `bun` | Bun runtime |
| `ultracite` | Ultracite linting |
| `validation` | Zod/Valibot/ArkType validation |

### Tools
| Skill | What It Does |
|-------|--------------|
| `domain-brainstormer` | Find available domain names |

---

## Creating Your Own Skills

When you accomplish something successfully, tell Claude:

```
"Create a skill from what we just did"
```

Claude packages the approach into a reusable skill. Next time you need the same pattern, it's applied automatically.

---

## For Vibe Coders

**You're the target audience.** This system is designed for product engineers who describe what they want in plain language.

### Tips

1. **Describe outcomes, not code**
   - "Add a dark mode toggle"
   - Not "Create a useState hook with..."

2. **One feature per session works best**
   - "Build the user profile page"
   - Not "Build the entire app"

3. **Point out inconsistencies**
   - "This doesn't match the dashboard"
   - Claude will check existing code and fix it

4. **Let Claude handle technical decisions**
   - You focus on what you want
   - Claude figures out how to build it

5. **Save your progress**
   - Say "save my progress" at session end
   - Or use `/log` slash command

---

## FAQ

**How do I start?**
Copy `.claude` to your project, open Claude Code, say "help me set up my project."

**Does this change my code?**
The skills system only adds files in `.claude`. Your app code changes as you request features.

**Can I use this with an existing project?**
Yes. Claude scans your codebase and auto-generates the projectbrief.

**How do I create new skills?**
Say "create a skill from what we just did" after accomplishing something you want to repeat.

**What about multi-session work?**
Say "save my progress" at session end. Claude saves context to `dev-context.md` so you pick up exactly where you left off.

**How does code review work?**
The `code-reviewer` skill guides Claude to review code for quality, security, and maintainability.

**How do I share skills with other projects?**
Copy `skills/`, `templates/`, and `commands/` folders. The `project/` folder stays unique to each codebase.

---

## Recommended Stack

For new web apps, the skills system recommends:

| Layer | Choice | Why |
|-------|--------|-----|
| Framework | TanStack Start | Modern full-stack React, no vendor lock-in |
| Database | Supabase | Managed Postgres, free tier, Realtime built-in |
| ORM | Drizzle | Lighter than Prisma, great for serverless |
| Auth | Better Auth | Flexible, server-side security, works with any DB |
| UI | Tailwind + shadcn/ui | Copy-paste components, easy customization |

The `project-setup` skill automatically recommends this stack and detects your package manager (bun, pnpm, npm, yarn).

---

## Changelog

### v1.1.0 (2026-02-04)
- **Server-side auth patterns** - Better Auth skill now enforces secure server-side authentication
- **TanStack Start guide** - Complete auth setup with `beforeLoad`, middleware, and `tanstackStartCookies`
- **tRPC authentication** - Protected procedures with session context
- **Supabase Realtime** - Real-time subscriptions guide with React hooks
- **Electric SQL guidance** - Clarified when to use (self-hosted only, not with Supabase)
- **Package manager detection** - Auto-detects bun/pnpm/yarn/npm from lock files
- **Recommended stack** - Default recommendation is now TanStack Start + Supabase + Drizzle + Better Auth
- **Update stack skill** - New skill to safely update all dependencies with breaking change detection
- **Railway deployment** - Complete deployment guide for TanStack Start with Nitro SSR, including troubleshooting for common issues

### v1.0.1 (2026-01-17)
- **Folder reorganization** - Project-specific files moved to `project/` subfolder
- **Feature planner skill** - Plan features with ASCII mockups before implementation
- **Easy skill sharing** - Copy `skills/`, `templates/`, `commands/` to new projects

### v1.0.0 (2026-01-04)
- **Skills-based architecture** - Technology patterns auto-trigger based on context
- **Workflow skills** - Code review, debugging, and linting built-in
- **Dev context** - Preserves state across sessions for complex features
- **Skill creator** - Package successful patterns for reuse
- **Verification patterns** - Each skill includes how to verify it works
- **Slash commands** - `/log` for explicit session logging

---

**Ready to start?** Copy `.claude` to your project and say "help me set up."
