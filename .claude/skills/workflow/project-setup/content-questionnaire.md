# Content Website Setup Questionnaire

## Reference for Claude

This questionnaire guides content website setup (Payload CMS + Astro). It's triggered when the user selects "Content website" in the project type question.

### Key Principle

The content website stack is fixed - Astro frontend, Payload CMS backend, Tailwind for styling. **The template code is already in place** - the project folder already contains the Astro frontend and Payload CMS backend as a monorepo. The wizard does NOT need to scaffold or create any project code files. It only needs to:

1. Ask deployment and description questions
2. Create project documentation files (projectbrief, tasks, changelog, dev-context)
3. Update environment variables based on the user's hosting choices

### Pre-Existing Project Brief

**Before starting the questionnaire, check if `project/projectbrief.md` already has real content.**

- **If projectbrief has technical content** (collections, routes, components, design tokens, etc.):
  1. Read it first - this is the template's technical reference
  2. Preserve all existing content - treat it as the project's foundation
  3. Only add/update the sections covered by the questionnaire (deployment, description, relevant skills)
  4. Merge, don't overwrite - append new sections below the existing content or update specific sections in place

- **If projectbrief is empty or has only template placeholders:**
  - Generate a new projectbrief from the template as usual

**Why this matters:** Content website templates often ship with a pre-written projectbrief documenting collections, routes, components, CSS tokens, and API patterns. This technical reference is fixed and should not be replaced by the wizard's slim output.

### How This Flow Gets Triggered

This questionnaire is reached in two ways:

1. **Auto-detected** - The wizard scans the project folder, finds Astro + Payload code, and jumps straight here. No project type question needed.
2. **User selected** - The user chose "Content website" from the project type question (empty folder scenario).

In both cases, the stack is already known. Only deployment and description questions are asked.

### Formatting Rules

1. **One question at a time** - Never ask multiple questions in one message
2. **Wait for answers** - Don't proceed until user responds
3. **Use plain English** - No technical jargon
4. **Offer recommendations** - Always suggest a best option
5. **Auto-detect project name** - Use the parent folder name

### Handling Pre-Filled Answers (from External Brief)

If the user pasted an external brief (see [brief-import.md](brief-import.md)), the project description is likely already covered. For content websites, the external brief is especially valuable because it often contains design system details, content structure, page layouts, and brand guidelines that go directly into the projectbrief.

1. **Q1-Q3 (Deployment):** Always ask - these are SkillKit-specific and won't be in the brief
2. **Q4 (Project description):** Skip if the brief already describes the website
3. **Q5 (Code review):** Always ask - SkillKit-specific

**Triple merge for content websites:** When there's a template projectbrief, an external brief, AND questionnaire answers, merge with this priority: External brief > Template projectbrief > Questionnaire answers. See [brief-import.md](brief-import.md) for full merge rules.

### Question Flow

#### Question 1: Backend Deployment (Payload CMS)

```
Where do you want to deploy the backend (your content management system)?

A. Cloudflare Workers (Recommended - $5/month, includes D1 database + R2 storage)
B. Railway ($5/month, built-in Postgres)
C. Vercel (free tier available)
D. Render
E. VPS (own server)
```

**Why Cloudflare Workers is recommended:** Everything under one roof with Cloudflare Pages for the frontend. $5/month Workers paid plan (Payload exceeds the free tier's 10MB limit), but D1 database and R2 media storage are included. If the user also deploys the frontend on Cloudflare Pages, they manage everything from one dashboard.

**If Railway is selected:** Read [railway-payload-deploy.md](railway-payload-deploy.md) for the complete step-by-step deployment flow (project creation, Postgres setup, GitHub connection, monorepo paths, environment variables).

---

#### Question 2: Database

```
What database do you want to use for your content?

A. PostgreSQL (Recommended - most common, works everywhere)
B. SQLite / D1 (lightweight, great for Cloudflare)
C. MongoDB
```

**Why PostgreSQL is recommended:** Battle-tested, works with every deployment platform, best Payload support. If the user chose Cloudflare Workers for the backend, mention that D1 (SQLite) is the natural pairing but Postgres via Supabase or Neon also works.

**Database hosting depends on backend choice + database type:**

| Backend | PostgreSQL | SQLite / D1 | MongoDB |
|---------|-----------|-------------|---------|
| Cloudflare Workers | Supabase / Neon (external) | D1 (built-in) | MongoDB Atlas (external) |
| Railway | Railway Postgres (built-in) | Local file | MongoDB Atlas (external) |
| Vercel | Supabase / Neon (external) | Local file | MongoDB Atlas (external) |
| Render | Render Postgres (built-in) | Local file | MongoDB Atlas (external) |

**If user picks Cloudflare Workers + PostgreSQL:** Ask a follow-up:

```
Where do you want to host your PostgreSQL database?

A. Supabase (Recommended - free tier, managed Postgres)
B. Neon (serverless Postgres)
C. I have my own database
```

---

#### Question 3: Frontend Deployment (Astro)

```
Where do you want to deploy the website itself (the pages visitors see)?

A. Cloudflare Pages (Recommended - free, fast, global CDN)
B. Netlify
C. Vercel
```

**Why Cloudflare Pages is recommended:** Free tier with unlimited bandwidth, global CDN, fast builds. If the backend is also on Cloudflare, everything is managed from one dashboard.

**If Cloudflare Pages is selected:** Read [cloudflare-pages-deploy.md](cloudflare-pages-deploy.md) for the complete step-by-step deployment flow (GitHub connection, build settings, environment variables, verification, and troubleshooting).

---

#### Question 4: Project Description

```
In 2-3 sentences, describe the website you're building.

(Example: "A portfolio website for a design agency, showcasing their projects, team members, and blog posts")
```

---

#### Question 5: Code Review (same as web app flow)

```
Would you like a second AI to review your code before it goes live?

A. Yes, set up Greptile (Recommended)
B. No, skip code review

Here's why I recommend it: I write your code, but I can make mistakes -
wrong patterns, security issues, or bugs I don't catch. Greptile is a
separate AI that reviews every pull request and gives it a confidence
score from 1 to 5. If the score is high (4 or 5), your code merges
automatically. If the score is low, I read Greptile's feedback, fix
the issues, and resubmit - all without you needing to understand the code.

It's like having a senior developer double-check everything before it ships.
Used by teams at NVIDIA, Brex, and Coinbase.

Free plan available at: https://greptile.com
```

**If Greptile is selected:** Add a `## Code Review` section to `project/dev-context.md`:

```markdown
## Code Review

Provider: Greptile
Auto-merge threshold: 4/5
Max fix cycles: 3
```

---

### After Questionnaire

**If projectbrief.md already has content:** Update it in place - add the deployment section, update the project description, and append the relevant skills mapping. Do not regenerate or overwrite existing technical sections.

**If projectbrief.md is empty:** Generate a new projectbrief using the template field mapping below.

Present the setup summary:

```
Here's your content website setup:

**Project:** [folder-name]
**Type:** Content Website

**Stack (already in your template):**
- Astro - Static site generator (fast, SEO-friendly)
- Payload CMS - Content management backend (admin panel, media, forms)
- Tailwind CSS - Styling

**Deployment:**
- Backend (Payload): [selected platform]
- Frontend (Astro): [selected platform]

**Skills I'll use:**
- astro - for page structure and components
- payload - for CMS configuration and content types

Your project code is already in place. I'll set up the project documentation and
configure environment variables for your chosen hosting platforms.

Does this look good?
```

**Important:** Do NOT offer to create or scaffold any project files. The Astro + Payload monorepo template is already in the project folder. The wizard only creates `.claude/project/` documentation files and updates `.env` files.

### Template Field Mapping

| Question | Template Section |
|----------|-----------------|
| Project type (content website) | `## Project Overview` -> "Type: Content Website" |
| Q1: Payload deployment | `## Deployment` -> Backend |
| Q2: Database | `## Tech Stack` -> Database |
| Q3: Frontend deployment | `## Deployment` -> Frontend |
| Q4: Project description | `## Project Overview` -> "What we're building" |
| Q5: Code review | `## Code Review` in dev-context.md (if Greptile) |

### Auto-Filled Sections

These sections are filled automatically (user doesn't need to answer):

| Section | Value |
|---------|-------|
| Framework | Astro |
| CMS | Payload |
| UI & Styling | Tailwind CSS |
| Auth | Payload built-in (admin users only) |
| SEO | Payload SEO plugin (pre-configured in template) |

### Relevant Skills Mapping

```markdown
## Relevant Skills

Based on your tech stack, reference these skills for implementation patterns:

| Technology | Skill Location |
|------------|----------------|
| Framework (Astro) | `.claude/skills/framework/astro/` |
| CMS (Payload) | `.claude/skills/cms/payload/` |
```

### Sections to Skip

Content websites don't need these projectbrief sections:
- Authentication (beyond Payload admin)
- Payments
- AI Integration
- Real-time features
- API Endpoints (Payload handles this)
- Database Schema (Payload handles this)

### Environment Variables

Content websites only need a `.env` file - there is no `.env.production`. All database connections point to external services (never local), so the same `.env` structure works for development. Production environment variables are configured directly in the deployment platform's dashboard (Cloudflare, Railway, Vercel, etc.).

The wizard creates `.env` files directly based on the user's database and deployment choices. There are no `.env.example` files to copy from.

#### Admin Seeding Variables

Before writing `.env` files, check if the user's auto-memory (`MEMORY.md`) has a `## Payload Admin Seeding` section. If it does:

1. Read the email templates and default password from memory
2. Replace `{project}` in each email template with the lowercase project folder name (no spaces, e.g., folder `TopElektro` becomes `topelektro`)
3. Write the resolved values into the project's `.env` as `PAYLOAD_ADMIN_EMAILS` (comma-separated) and `PAYLOAD_ADMIN_PASSWORD`
4. Add an `## Admin Seeding` section to `dev-context.md` documenting what was configured:

```markdown
## Admin Seeding

Admin accounts are auto-created on first Payload startup via the `onInit` hook.

| Variable | Value |
|----------|-------|
| `PAYLOAD_ADMIN_EMAILS` | resolved-email1@example.com,resolved-email2@example.com |
| `PAYLOAD_ADMIN_PASSWORD` | (set in .env) |

To change, update `.env` and clear the database users collection.
```

If no admin seeding config exists in memory, skip these variables entirely. The user will create admin accounts manually via the Payload registration UI on first visit to `/admin`.

#### PostgreSQL Setup

**`.env`:**
```
# App
APP_URL=http://localhost:4321

# Payload CMS
PAYLOAD_URL=http://localhost:3000
PAYLOAD_SECRET=  # Generate: openssl rand -base64 32
DATABASE_URI=  # From your database provider (see table below)
PAYLOAD_ADMIN_EMAILS=  # Comma-separated admin emails (from dev-context.md admin seeding config)
PAYLOAD_ADMIN_PASSWORD=changeme123  # Password for auto-seeded admin accounts
```

**Auto-seeded admin accounts:** On first startup, Payload automatically creates admin users for each email in `PAYLOAD_ADMIN_EMAILS`. All accounts use `PAYLOAD_ADMIN_PASSWORD` as the password. Seeding only runs when the users collection is empty.

**Where to get DATABASE_URI:**

| Backend + DB Provider | DATABASE_URI source |
|----------------------|---------------------|
| Railway + Railway Postgres | Auto-provided by Railway |
| Cloudflare Workers + Supabase | From Supabase dashboard -> Settings -> Database |
| Cloudflare Workers + Neon | From Neon dashboard |
| Vercel + Supabase | From Supabase dashboard |
| Render + Render Postgres | Auto-provided by Render |

#### SQLite / D1 Setup

**`.env`:**
```
# App
APP_URL=http://localhost:4321

# Payload CMS
PAYLOAD_URL=http://localhost:3000
PAYLOAD_SECRET=  # Generate: openssl rand -base64 32
PAYLOAD_ADMIN_EMAILS=  # Comma-separated admin emails (from dev-context.md admin seeding config)
PAYLOAD_ADMIN_PASSWORD=changeme123  # Password for auto-seeded admin accounts
```

Locally, Payload uses a SQLite file - no DATABASE_URI needed. D1 is configured via `wrangler.toml` bindings for production.

#### MongoDB Setup

**`.env`:**
```
# App
APP_URL=http://localhost:4321

# Payload CMS
PAYLOAD_URL=http://localhost:3000
PAYLOAD_SECRET=  # Generate: openssl rand -base64 32
MONGODB_URI=  # From MongoDB Atlas (connection string)
PAYLOAD_ADMIN_EMAILS=  # Comma-separated admin emails (from dev-context.md admin seeding config)
PAYLOAD_ADMIN_PASSWORD=changeme123  # Password for auto-seeded admin accounts
```

#### Platform-Specific Additions

| Platform (Payload) | Extra Variables |
|---------------------|-----------------|
| Cloudflare Workers | D1/R2 bindings in `wrangler.toml` |
| Railway | `PORT=3000` (auto-set by Railway) |
| Vercel | None (serverless, no PORT needed) |
| Render | `PORT=3000` |

**Frontend platforms (Cloudflare Pages / Netlify / Vercel):** No extra variables needed for static Astro sites.

### Initial Tasks

Generate these initial tasks in `project/tasks.md`. The checklist is numbered and ordered - follow it top to bottom.

Since the template code is already in place, tasks focus on getting running and deploying. Replace `[selected platform]` with the user's actual choices from the questionnaire.

```markdown
## Setup Checklist

Follow these steps in order. Each step depends on the previous ones.

### Phase 1: Local Setup
- [ ] 1. Update project name in package.json files (root, backend/, frontend/) - change the `name` field to match the folder name
- [ ] 2. **Update DATABASE_URI** in backend/.env - the template's value points to the template's own database, change it to the new project's Postgres URL
- [ ] 3. Delete stale node_modules and reinstall dependencies (`rm -rf node_modules && bun install` in both backend/ and frontend/)
- [ ] 4. Start dev servers (backend: `pnpm run dev` on :3000, frontend: `bun run dev` on :4321)
- [ ] 5. Verify admin panel at localhost:3000/admin (auto-seeded admin accounts should work)
- [ ] 6. Verify frontend at localhost:4321

### Phase 2: GitHub
- [ ] 7. Create GitHub repository
- [ ] 8. First commit and push to main

### Phase 3: Backend Deployment ([selected platform])
- [ ] 9. Create [selected platform] project and provision PostgreSQL database
- [ ] 10. Create backend service, connect to GitHub repo, set root directory to /backend
- [ ] 11. Set environment variables (DATABASE_URI, PAYLOAD_PUBLIC_SERVER_URL, PAYLOAD_SECRET)
- [ ] 12. Generate domain, deploy, and verify admin panel works at production URL

### Phase 4: Frontend Deployment (Cloudflare Pages)
- [ ] 13. Connect Cloudflare Pages to GitHub repo
- [ ] 14. Configure build settings: command `bun install && bun run build`, output `dist`, root `frontend`
- [ ] 15. Set PAYLOAD_URL environment variable (production backend URL from step 12)
- [ ] 16. Save and Deploy, verify site loads at pages.dev URL

### Phase 5: Final Verification
- [ ] 17. Admin panel works at production URL with seeded credentials
- [ ] 18. Frontend fetches and displays content from production backend
```

**Notes:**
- There is no "Start the database" task. Content websites use external database services (Railway Postgres, Supabase, Neon, etc.), not local Docker containers.
- Steps 3-5 (local dev) are optional before deployment but recommended to verify the template works.
- The backend must be deployed before the frontend because Cloudflare Pages needs `PAYLOAD_URL`.
- For detailed commands, see the Railway guide (`railway-payload-deploy.md`) and Cloudflare Pages guide (`cloudflare-pages-deploy.md`).

### Getting Started Guide

After the setup wizard completes, walk the user through getting the project running locally. Use the project's detected package manager (replace `[pm]` with bun/npm/pnpm/yarn).

#### Step 1: Set Up Environment Files

Create a `.env` file in both backend and frontend based on the user's database and deployment choices. There are no `.env.example` files to copy from - the wizard generates `.env` files directly using the templates in the Environment Variables section above.

```
Backend:
- Create backend/.env with the appropriate variables for the chosen database
- Generate a random PAYLOAD_SECRET (this secures the admin panel)
- DATABASE_URI left empty for user to fill in from their provider (PostgreSQL)
- Or no DATABASE_URI needed (SQLite/D1 - uses local file)

Frontend:
- Create frontend/.env with APP_URL=http://localhost:4321
- Defaults are fine for local development
```

**Claude should do this for the user** - generate the `.env` files directly with a random PAYLOAD_SECRET and the correct variables for the chosen database type.

**No `.env.production` for content websites.** Production environment variables are configured directly in the deployment platform's dashboard (Cloudflare, Railway, Vercel, etc.). The `.env` file is only for local development.

#### Step 2: Install Dependencies

The template's `node_modules` are stale from the copy - always remove them before installing fresh.

```
cd backend && rm -rf node_modules && [pm] install
cd frontend && rm -rf node_modules && [pm] install
```

Or if the root has a workspace setup, `rm -rf node_modules && [pm] install` from the root.

#### Step 3: Start Both Dev Servers

Two terminal windows needed:

```
Terminal 1 (backend):  cd backend && pnpm run dev    -> Payload admin on localhost:3000
Terminal 2 (frontend): cd frontend && bun run dev    -> Astro on localhost:4321
```

**Important:** The backend uses `pnpm run dev` (not bun) because Payload requires Node.js as its runtime. Bun works as a package manager for installing dependencies, but the dev server must run through pnpm/Node.js. The frontend (Astro) has full Bun support.

Or from the root folder using convenience scripts: `bun run dev:backend` and `bun run dev:frontend` (these delegate to the correct package managers).

#### Step 4: Log Into Admin Panel

Open `localhost:3000/admin` in the browser.

**If admin seeding is configured** (dev-context.md has `## Admin Seeding`): Admin accounts were automatically created on first startup using the emails from `PAYLOAD_ADMIN_EMAILS`. Log in with any of those emails and the password from `PAYLOAD_ADMIN_PASSWORD` (default: `changeme123`).

**If no admin seeding:** Payload will show a registration form to create the first admin user.

#### Step 5: Start Adding Content

Once logged in, the sidebar shows all collections (Posts, Pages, Categories, Tags, Media). Start creating categories and tags, then write posts and pages.

**Communication tip:** Walk the user through these steps one at a time after setup completes. Don't dump all steps at once - ask if they're ready for the next step after each one completes.
