# Project Setup Wizard Flow

## Overview

This document defines the complete wizard experience for setting up new projects or documenting existing codebases. The wizard guides users through a conversational flow, one question at a time, and generates all necessary documentation files.

## Flow Diagram

```
User starts conversation
        ↓
Check environment:
├── project/projectbrief.md empty/missing?
├── project/projectbrief.md has content? (pre-filled template reference)
├── Code files exist?
└── package.json present?
        ↓
Route to appropriate flow:
├── Has Astro + Payload code → Content Website Flow (auto-detected, skip project type question)
│   └── projectbrief.md has content? → Read it, preserve it, merge questionnaire answers in
├── Has other code → Existing Codebase Flow
├── Empty folder → Ask Project Type Question
│   ├── Web application → New Web App Flow
│   └── Content website → Content Website Flow
└── Has projectbrief → Analyze & Recommend
```

**Auto-detection for content websites:** Before asking any questions, check if the project folder contains both Astro and Payload. Look for `astro` + `payload` in package.json dependencies, or `astro.config.*` + `payload.config.ts` files, or a monorepo with `backend/` and `frontend/` folders containing these. If detected, skip the project type question and go straight to the Content Website Flow (deployment questions only).

---

## New Project Flow

### Step 1: Welcome & Project Type

**What to do:**
- Greet the user warmly
- Detect project name from folder
- Ask the project type question (this is always the first question)

**Example output:**
```
Welcome! I'll help you set up your project.

Project name: **my-awesome-app** (from your folder)

First question: What type of project are you building?

A. Web application (interactive app with user accounts, database, etc.)
B. Content website (blog, marketing site, portfolio - powered by CMS)
```

**Routing:**
- **Web application** → Continue with Step 2 below (web app questionnaire)
- **Content website** → Jump to [Content Website Flow](#content-website-flow)

### Step 2: Web App Questionnaire

**Reference:** [questionnaire.md](questionnaire.md)

**Rules:**
- Ask ONE question at a time
- Wait for user's answer before proceeding
- Offer recommendations when user is unsure
- Use plain language, no technical jargon

**Question flow:**
1. What are you building? (2-3 sentences)
2. Who will use this?
3. What type of application? (Web/Mobile/Both/Extension/Plugin)
4. What features do you need? (Login, database, payments, etc.)
5. Experience level? (Affects explanation depth)
6. Technology preferences? (Or let Claude recommend)
7. Timeline? (Exploring/Side project/Launch soon/Urgent)

### Step 3: Present Recommendations

**What to do:**
- Summarize user's answers
- Present recommended tech stack
- Explain WHY each choice (one line each)
- List which skills will be used
- Ask for approval before proceeding

**Example output:**
```
Based on your answers, here's what I recommend:

**Project:** Task Manager
**Type:** Web Application

**Recommended Setup:**
- Next.js 14 - Modern React framework, great for web apps
- Prisma + Neon - Simple database setup that scales
- Better Auth - Easy authentication with social login
- Shadcn UI - Beautiful, accessible components
- Tailwind CSS - Fast styling without writing CSS

**Skills I'll use:**
- nextjs - for routing and server components
- prisma - for database queries
- better-auth - for login/signup
- shadcn - for UI components

Does this look good? I can explain any choice or suggest alternatives.
```

### Step 4: Create Documentation Files

**Reference:** [questionnaire.md](questionnaire.md) → Template Field Mapping

**Process:**
1. Read template from `.claude/templates/`
2. Populate with user's answers
3. Skip sections that don't apply
4. Write to target location

**Files to create:**

| File | Template | When |
|------|----------|------|
| `.claude/project/projectbrief.md` | projectbrief-template.md | Always |
| `.claude/project/tasks.md` | tasks-template.md | Always |
| `.claude/project/changelog.md` | changelog-template.md | Always |
| `/docs/prd.md` | prd-template.md | If user wants detailed planning doc |

### Step 5: Generate Environment Files

**Reference:** [env-vars-template.md](../../../templates/env-vars-template.md)

Create `.env` and `.env.production` based on the user's confirmed tech stack:

1. Read the env-vars-template
2. Include only the service groups matching the user's confirmed stack
3. Apply the framework prefix map (e.g., `[CLIENT] APP_URL` becomes `VITE_APP_URL` for TanStack Start)
4. Write `.env` with localhost defaults pre-filled where applicable
5. Write `.env.production` with empty values and comments showing where to find each value
6. Ensure `.gitignore` includes `.env`, `.env.local`, `.env.production`, `.env.*.local`

**Example output:**
```
I've created your environment files:

- .env - Local development (localhost defaults filled in)
- .env.production - Production template with instructions for each value
- .gitignore updated

Fill in your API keys as you set up each service. The production file shows exactly where to find each value.
```

### Step 6: Map Skills to Tech Stack

**What to do:**
- Add "Relevant Skills" section to project/projectbrief.md
- Map each technology to its skill location

**Example in project/projectbrief.md:**
```markdown
## Relevant Skills

Based on your tech stack, reference these skills for implementation patterns:

| Technology | Skill Location |
|------------|----------------|
| Framework (Next.js) | `.claude/skills/framework/nextjs/` |
| Database (Prisma) | `.claude/skills/database/prisma/` |
| Auth (Better Auth) | `.claude/skills/auth/better-auth/` |
| UI (Shadcn) | `.claude/skills/ui/shadcn/` |
```

### Step 7: Complete Setup

**What to show:**
- Confirmation that files were created
- Summary of what was configured
- Clear next steps
- Offer to start building

**Example output:**
```
Your project is now set up!

**Created:**
- .claude/project/projectbrief.md - Your project overview
- .claude/project/tasks.md - Task tracking (empty, ready for features)
- .claude/project/changelog.md - Session history (first entry added)
- .env - Local development environment variables
- .env.production - Production environment template (copy-paste to Railway/Vercel)
- .gitignore updated - Environment files excluded from git

**Tech Stack:** TanStack Start + Supabase + Better Auth + Shadcn

**Next steps:**
1. Review project/projectbrief.md to make sure it looks right
2. Fill in API keys in .env as you set up each service
3. Tell me what feature you want to build first!

What would you like to work on?
```

---

## Content Website Flow

The content website flow is for projects using Astro + Payload CMS. **The template code is already in the project folder** (Astro frontend + Payload backend monorepo). The wizard only creates documentation and configures environment variables.

**Reference:** [content-questionnaire.md](content-questionnaire.md)

### Step 1: Content Website Questionnaire

Ask these questions one at a time:

1. **Backend deployment** - Where to deploy Payload CMS? (Cloudflare Workers [recommended] / Railway / Vercel / Render / VPS)
2. **Database** - What database for content? (PostgreSQL [recommended] / SQLite-D1 / MongoDB)
3. **Frontend deployment** - Where to deploy the Astro website? (Cloudflare Pages [recommended] / Netlify / Vercel)
4. **Project description** - What is this website about? (free text)
5. **Code review** - Set up Greptile? (Yes / No)

### Step 2: Present Summary

Show the user what will be configured:

```
Here's your content website setup:

**Project:** [folder-name]
**Type:** Content Website

**Stack (already in your template):**
- Astro - Static site generator
- Payload CMS - Content management backend
- Tailwind CSS - Styling

**Deployment:**
- Backend (Payload): [selected platform]
- Database: [selected database]
- Frontend (Astro): [selected platform]

Your project code is already in place. I'll set up the project documentation
and configure environment variables for your chosen hosting platforms.

Does this look good?
```

### Step 3: Create Documentation Files

**Check projectbrief.md first:**

- **If projectbrief.md already has content** (technical reference with collections, routes, components, etc.): Read it, preserve all existing content, and only add/update the sections from the questionnaire (deployment, description, relevant skills). Do NOT overwrite or regenerate the file.
- **If projectbrief.md is empty or missing:** Generate a slim projectbrief from the template.

| File | Content |
|------|---------|
| `project/projectbrief.md` | If pre-filled: merge in deployment + description. If empty: project type, description, fixed stack (Astro + Payload + Tailwind), deployment targets, relevant skills |
| `project/dev-context.md` | Package manager, code review config |
| `project/tasks.md` | Configuration and deployment tasks (no scaffolding tasks - template is ready) |
| `project/changelog.md` | Setup session entry |

**Sections to skip in projectbrief (when generating new):** Auth, Payments, AI Integration, Database Schema, API Endpoints, Real-time features (all handled by the Payload template).

### Step 4: Update Environment Variables

Update `.env` and `.env.production` based on the user's hosting choices. The template may already have default env files - check first and update rather than overwrite.

**See:** [content-questionnaire.md](content-questionnaire.md) → Environment Variables section

### Step 5: Complete Setup & Getting Started

Mark `.setup-complete`, then walk the user through getting the project running locally.

**See:** [content-questionnaire.md](content-questionnaire.md) -> Getting Started Guide

**Example output:**
```
Your content website is configured!

**Created:**
- .claude/project/projectbrief.md - Your project overview
- .claude/project/tasks.md - Task tracking
- .claude/project/changelog.md - Session history
- Environment variables updated for [Payload platform] + [Frontend platform]

**Your template includes:**
- Astro frontend with Tailwind
- Payload CMS with admin panel, SEO, media handling
- Pre-configured content types

Let's get your project running locally. First, I'll set up your environment files...
```

Then walk through the Getting Started steps one at a time:
1. Set up .env files (Claude does this automatically)
2. Start the database (if Postgres setup)
3. Install dependencies
4. Start both dev servers
5. Create admin account
6. Start adding content

---

## Existing Codebase Flow

### Step 1: Scan Codebase

**Reference:** [detection.md](detection.md)

**What to scan:**
- `package.json` - Dependencies
- File structure - Folders and patterns
- Config files - Framework configs
- Existing `.claude/` folder - Previous setup

### Step 2: Present Findings

**What to show:**
- Detected framework and version
- Database/ORM if found
- Auth system if found
- UI library if found
- Deployment config if found

**Example output:**
```
I analyzed your codebase. Here's what I found:

**Framework:** Next.js 14 (App Router)
**Database:** Prisma + PostgreSQL
**Authentication:** Better Auth
**UI:** Tailwind CSS + Shadcn UI
**Deployment:** Configured for Vercel

**Code Structure:**
- 12 pages/routes
- 24 components
- 5 database models
- 8 API endpoints

Is this correct? Let me know if I missed anything or got something wrong.
```

### Step 3: Gather Context

**Questions to ask:**
1. Confirm detected stack is correct
2. Ask what the project does (for projectbrief)
3. Ask what they want to work on

### Step 4: Create Documentation

Same as New Project Flow Step 4, but pre-populated with detected info.

### Step 5: Generate Environment Files

Same as New Project Flow Step 5, but with an extra check:

1. Check if `.env` already exists in the project root
2. If it exists: read it, compare with the recommended variables for the detected stack, and show the user which variables are missing. Ask if they want to add the missing ones or skip.
3. If it doesn't exist: generate both `.env` and `.env.production` as normal
4. Always check `.env.production` - if missing, generate it even if `.env` exists

This ensures existing projects get the production reference file without disrupting their current local setup.

### Step 6: Complete Setup

Same as New Project Flow Step 7.

---

## Edge Cases

### User has partial setup

If `.claude/project/projectbrief.md` exists but is incomplete:
- Read existing content
- Identify missing sections
- Ask questions to fill gaps
- Update file (don't overwrite)

### User wants to change recommendations

If user disagrees with a recommendation:
- Ask what they'd prefer
- Explain trade-offs briefly
- Accept their choice
- Update recommendations

### User is unsure about everything

If user answers "I don't know" to most questions:
- Pick sensible defaults for their project type
- Explain each choice briefly
- Proceed with defaults
- Tell them they can change later

### No code, no idea

If user has empty folder and no clear idea:
- Ask what problem they want to solve
- Suggest project types based on their answer
- Guide them to clarify their vision
- Then start normal questionnaire

---

## Templates Reference

All templates are in `.claude/templates/`:

| Template | Purpose | Size |
|----------|---------|------|
| `projectbrief-template.md` | AI context document | ~430 lines |
| `tasks-template.md` | Task tracking | ~78 lines |
| `changelog-template.md` | Session history | ~40 lines |
| `dev-context-template.md` | In-progress work | ~33 lines |
| `prd-template.md` | Human requirements doc | ~655 lines |
| `env-vars-template.md` | Env var mapping by service | ~200 lines |

**Note:** `dev-context.md` is created during setup with initial structure (Package Manager, Code Review sections). The session-log skill adds work-in-progress entries later.
