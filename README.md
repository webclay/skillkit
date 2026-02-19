# SkillKit

> A skills system for Claude Code that teaches it exactly how to build your app, so you never have to explain the same thing twice.

**Version:** 1.2.9
**Author:** Manuel Merz
**License:** MIT
**Date:** 19/02/2026

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
| `project/dev-context.md` | Architectural decisions, project config, in-progress state | Every conversation and when picking up unfinished work |

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
| `better-auth` | Better Auth with server-side protection, tRPC integration, TanStack Start patterns, plus official BetterAuth guides for security, 2FA, organizations, email/password |

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

### CMS
| Skill | What It Does |
|-------|--------------|
| `payload` | Payload CMS - headless backend with admin panel, blocks, media, REST API, Form Builder |

### API
| Skill | What It Does |
|-------|--------------|
| `server-actions` | Next.js Server Actions |
| `orpc` | oRPC type-safe APIs |
| `unkey` | Unkey API key management |
| `zefix` | Zefix Swiss company lookup (search, UID, SOGC publications) |
| `apollo` | Apollo.io people and company enrichment, search, and CRM |

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
| `autosend` | AutoSend transactional and marketing email (volume-based pricing, Resend drop-in compatible) |
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

### Design
| Skill | What It Does |
|-------|--------------|
| `typography` | Web typography rules - font sizes, line height, line length, font pairing, headings, spacing |

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

### v1.2.9 (2026-02-19)
- **Official BetterAuth skills** - Integrated 6 official skills from `better-auth/skills` into the Better Auth skill: best practices (config, sessions, hooks, plugins), security (rate limiting, CSRF, trusted origins, cookies, OAuth PKCE, IP tracking, audit logging), create-auth wizard (project scanning + questionnaire), email/password (verification, password reset, hashing, timing attack prevention), organizations (multi-tenant, teams, RBAC, invitations, dynamic access control), and two-factor auth (TOTP, OTP, backup codes, trusted devices)

### v1.2.8 (2026-02-17)
- **Content website env simplification** - Content websites no longer create `.env.production` (production vars go directly in deployment platform dashboards). Removed `.env.example` concept from content website wizard - Claude generates `.env` files directly. Database URLs point to external services by default.

### v1.2.7 (2026-02-17)
- **Content website setup fix** - Environment setup now creates both `.env` and `.env.production` files (was only creating `.env`). Removed incorrect "Start PostgreSQL via Docker Compose" task since content websites use external database services (Cloudflare D1, Supabase, Neon, Railway, etc.)

### v1.2.6 (2026-02-16)
- **External brief import** - Setup wizard now asks "Do you have a project brief?" before the questionnaire. Users can paste briefs from Claude Projects, Google Docs, or any source. The wizard extracts relevant info, skips already-answered questions, and merges everything into the internal projectbrief format. Works for both web apps and content websites.

### v1.2.5 (2026-02-16)
- **Pre-filled projectbrief support** - Content website setup wizard now reads and preserves existing projectbrief.md content instead of overwriting it, enabling templates to ship with pre-written technical references

### v1.2.4 (2026-02-16)
- **Typography skill** - Web typography rules based on Butterick's Practical Typography: font sizes (15-25px body), line height (120-145%), line length (45-90 chars), heading hierarchy, font pairing, color, letter spacing, kerning, paragraph spacing, responsive typography, and a complete starter stylesheet

### v1.2.3 (2026-02-15)
- **Content website setup wizard** - Project setup now asks "Web application or Content website?" as the first question, routing to the appropriate questionnaire
- **Content website questionnaire** - Simplified 3-question flow for Astro + Payload CMS sites: deployment targets (backend + frontend) and project description
- **Template-aware setup** - Content website wizard recognizes that the Astro + Payload monorepo template is already in place, only creates documentation and configures environment variables
- **Astro + Payload detection** - Codebase auto-detection now recognizes Astro and Payload CMS projects

### v1.2.2 (2026-02-14)
- **Payload on Cloudflare Workers** - Full deployment guide with D1 database, R2 storage, Wrangler config, migrations, and cost comparison (~$5/month vs ~$28/month Railway)
- **Astro + Payload CMS theming** - CMS-driven theme pattern using CSS variables generated from Payload fields via Astro's `<style define:vars>`
- **Payload build trigger strategies** - Three approaches for triggering Astro rebuilds: admin button, debounced auto-rebuild, and per-change webhooks
- **Payload live preview with Astro** - postMessage-based preview in Payload's side panel, plus block-level preview via server routes
- **Local API caveat** - Documented `__dirname` ES module scope error when using Payload's Local API with Astro SSG mode

### v1.2.1 (2026-02-13)
- **Payload CMS skill** - Comprehensive headless CMS skill with collections, blocks, media, hooks, REST API, Form Builder plugin, monorepo deployment (Railway + Cloudflare Pages), and Bun runtime compatibility
- **Astro headless CMS integration** - View Transitions, prefetching, and complete Payload + Astro patterns
- **Server-first rendering guide** - Anti-flash/double-load patterns in best-practices and TanStack Start skills
- **Loader caching** - staleTime patterns, TanStack Query ensureQueryData, and Link preloading for skeleton-only-on-first-visit

### v1.2.0 (2026-02-11)
- **Simplified workflow** - Consolidated /push + /merge into single `/wrap-up` command (log, lint, build, commit, push, PR, merge, cleanup)
- **Greptile code review** - Optional automated code review cycle with confidence scoring, auto-fix, and 100-file limit detection
- **Project config in dev-context.md** - Package Manager and Code Review config now live in `project/dev-context.md` (survives SkillKit updates)
- **CLAUDE.md stays generic** - No more project-specific sections in CLAUDE.md, safe to overwrite on updates
- **Dynamic package manager** - All skills use project-configured runner instead of hardcoded `npx`
- **UI scope discipline** - New rule preventing over-application of styles to unrelated components
- **TanStack Start security** - Nitro middleware for HTTP security headers (X-Content-Type-Options, X-Frame-Options, HSTS, etc.)
- **Database bundle leak prevention** - Prisma and Drizzle skills now enforce API route patterns to prevent client bundle leaks
- **AutoSend email skill** - Cheaper email alternative with transactional, marketing, bulk send, Better Auth integration, and Resend drop-in compatibility
- **Zefix API skill** - Swiss company data lookup via Zefix Public REST API (search, UID lookup, SOGC publications, legal forms)
- **Apollo API skill** - Apollo.io people and company data enrichment, search, and CRM operations (bulk enrichment, credit-free people search)

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
