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

**MANDATORY FIRST ACTION - Environment Reset:**
Before presenting the summary or creating any documentation files, the wizard MUST check if this is a duplicated template and reset environment files. See the "Template Environment Reset" section below. This is NOT optional - skipping it causes the new project to use the old project's R2 bucket, database, and deploy hooks.

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

**Important:** ALWAYS include dependency install commands before dev server commands in any output. When the wizard presents "how to run" or "next steps", the install step (`[pm] install` in each folder) MUST come before the dev server commands. Users duplicating the template won't have node_modules installed.

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

### Template Environment Reset

**CRITICAL - MUST RUN BEFORE ANYTHING ELSE IN SETUP.**

When Astro + Payload code is auto-detected, the project is almost certainly a duplicated template. The wizard MUST reset environment files IMMEDIATELY after the questionnaire, BEFORE creating documentation files or presenting the getting started guide. Skipping this step causes the new project to upload media to the old project's R2 bucket and connect to the old project's database.

**This is a BLOCKING step** - do not proceed with documentation creation or getting started guide until env files are reset.

#### Detection

Check if `backend/.env` exists and contains template-specific values (non-empty `R2_ACCESS_KEY_ID`, `CLOUDFLARE_DEPLOY_HOOK_URL`, or `DATABASE_URI` pointing to a remote host). If so, this is a duplicated template that needs resetting. **When in doubt, always reset** - it's safer to clear values than to leave old project credentials in place.

#### Reset Actions

**Auto-generate (Claude does this):**
- `PAYLOAD_SECRET` - Generate a new one: `openssl rand -hex 32`

**Keep unchanged:**
- `PAYLOAD_ADMIN_EMAILS` - Read from user's auto-memory (`## Payload Admin Seeding` section)
- `PAYLOAD_ADMIN_PASSWORD` - Read from user's auto-memory

**Clear to empty (user fills in later):**
- `DATABASE_URI` - New project needs its own database
- `R2_BUCKET` - New project needs its own R2 bucket
- `R2_ENDPOINT` - Tied to Cloudflare account (may be same, but user should confirm)
- `R2_PUBLIC_URL` - Tied to new R2 bucket
- `R2_ACCESS_KEY_ID` - New API token recommended per project
- `R2_SECRET_ACCESS_KEY` - New API token recommended per project
- `CLOUDFLARE_DEPLOY_HOOK_URL` - New Cloudflare Pages project = new hook

**Set to localhost defaults (`.env` only):**
- `PAYLOAD_PUBLIC_SERVER_URL` = `http://localhost:3000`
- `CLIENT_URI` = `http://localhost:4321`

**Clear in `.env.production`:**
- Same variables as above, but `PAYLOAD_PUBLIC_SERVER_URL` left empty (set via Railway dashboard)
- No `CLIENT_URI` in production

#### Guided Checklist

After resetting env files, present the user with a checklist of services to create. Show this as a step-by-step guide, one item at a time:

```
Your environment files have been reset for the new project.
I've auto-generated a new PAYLOAD_SECRET and kept your admin accounts.

Here's what you need to set up (I'll walk you through each one):

1. **PostgreSQL Database** - Create a new database (Railway, Supabase, or Neon)
   → Gives you: DATABASE_URI

2. **Cloudflare R2 Bucket** - Create a new R2 bucket in Cloudflare dashboard
   → Gives you: R2_BUCKET, R2_ENDPOINT, R2_PUBLIC_URL

3. **R2 API Token** - Create an R2 API token with Object Read & Write
   → Gives you: R2_ACCESS_KEY_ID, R2_SECRET_ACCESS_KEY

4. **Cloudflare Pages** - Connect your GitHub repo to Cloudflare Pages
   → Gives you: CLOUDFLARE_DEPLOY_HOOK_URL

Let's start with step 1. Do you have a database ready, or should I help you set one up?
```

Walk the user through each step one at a time. When they provide a value (like a DATABASE_URI), write it into both `.env` and `.env.production` (where applicable) immediately.

### Environment Variables

Content websites use both `.env` (local development) and `.env.production` (Railway deployment reference). Production environment variables are also configured in the deployment platform's dashboard, but `.env.production` serves as the reference file committed to the repo with placeholder values.

The wizard creates/resets `.env` files directly based on the user's database and deployment choices. There are no `.env.example` files to copy from.

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

**WARNING about `$` in passwords:** Next.js uses `dotenv-expand`, which silently interprets `$` as variable references. A password like `Plok$3ad*099` becomes `Plok` (everything after `$3` expands to empty string). Always escape `$` with `\$` in `.env` files: `Plok\$3ad*099`. When writing passwords to `.env`, auto-escape any `$` characters.

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
- [ ] 1. **Reset environment files** - Clear template-specific values from `.env` and `.env.production`, auto-generate new PAYLOAD_SECRET, keep admin accounts
- [ ] 2. Update project name in package.json files (root, backend/, frontend/) - change the `name` field to match the folder name
- [ ] 3. **Set up PostgreSQL database** - Create a new database (Railway/Supabase/Neon) and add DATABASE_URI to `.env`
- [ ] 4. **Set up Cloudflare R2** - Create R2 bucket, enable public access, create API token, add all R2 credentials to `.env` (R2_BUCKET, R2_ENDPOINT, R2_PUBLIC_URL, R2_ACCESS_KEY_ID, R2_SECRET_ACCESS_KEY)
- [ ] 5. Delete stale node_modules and reinstall dependencies (`rm -rf node_modules && [pm] install` in both backend/ and frontend/)
- [ ] 6. **Run database migrations** - `cd backend && pnpm payload migrate` (creates all tables in the fresh database - MUST happen before starting servers)
- [ ] 7. Start dev servers (backend: `pnpm run dev` on :3000, frontend: `[pm] run dev` on :4321)
- [ ] 8. Verify admin panel at localhost:3000/admin (auto-seeded admin accounts should work)
- [ ] 9. Upload a test image - verify it stores in R2 (not locally)
- [ ] 10. Verify frontend at localhost:4321

### Phase 2: GitHub & Git Workflow
- [ ] 11. Create GitHub repository
- [ ] 12. First commit and push to main
- [ ] 13. Create `staging` branch from main (`git checkout -b staging && git push -u origin staging`)

### Phase 3: Backend Deployment ([selected platform])
- [ ] 14. Create [selected platform] project and provision PostgreSQL database
- [ ] 15. Create backend service, connect to GitHub repo, set root directory to /backend
- [ ] 16. Set environment variables (DATABASE_URI, PAYLOAD_PUBLIC_SERVER_URL, PAYLOAD_SECRET, R2 keys)
- [ ] 17. Generate domain, deploy, and verify admin panel works at production URL

### Phase 4: Frontend Deployment (Cloudflare Pages)
- [ ] 18. Connect Cloudflare Pages to GitHub repo
- [ ] 19. Configure build settings: command `bun install && bun run build`, output `dist`, root `frontend`
- [ ] 20. Set environment variables: Production (`main`) gets production PAYLOAD_URL, Preview (`staging`) gets staging PAYLOAD_URL
- [ ] 21. Create deploy hook, add CLOUDFLARE_DEPLOY_HOOK_URL to backend `.env` and `.env.production`
- [ ] 22. Save and Deploy, verify site loads at pages.dev URL
- [ ] 23. Verify staging deploys automatically at `staging.{project}.pages.dev` (push to staging branch to test)

### Phase 5: CI & Performance Monitoring
- [ ] 24. Add `PAYLOAD_URL` as a GitHub Actions secret (pointing to a reachable Payload instance)
- [ ] 25. Verify Lighthouse CI runs on PRs to main/staging (`.github/workflows/lighthouse.yml`)
- [ ] 26. Verify Performance Budget checks run on PRs (`.github/workflows/performance-budget.yml` - JS < 50KB, HTML < 200KB)

### Phase 6: Final Verification
- [ ] 27. Admin panel works at production URL with seeded credentials
- [ ] 28. Upload a test image in production - verify it stores in R2 and displays on frontend
- [ ] 29. Frontend fetches and displays content from production backend
- [ ] 30. Staging site works at `staging.{project}.pages.dev` with noindex robots.txt
```

**Notes:**
- There is no "Start the database" task. Content websites use external database services (Railway Postgres, Supabase, Neon, etc.), not local Docker containers.
- **R2 must be set up before starting dev servers** (step 4 before step 7). Without R2 credentials, the s3Storage plugin is disabled and uploads go to local disk.
- **Migrations must run before starting dev servers** (step 6 before step 7). The fresh database has no tables - starting the backend without migrations causes errors because Payload queries non-existent tables. This creates a problem: files on local disk won't exist in production, and re-uploading everything to R2 later is tedious. Set up R2 early so all uploads go there from the start.
- The backend must be deployed before the frontend because Cloudflare Pages needs `PAYLOAD_URL`.
- **Staging branch** - Create before deploying. Cloudflare Pages auto-deploys all branches: `main` to production, `staging` to `staging.{project}.pages.dev`. Set different env vars per branch in Cloudflare dashboard (Settings > Environment variables > Production vs Preview).
- **CI workflows** - Lighthouse CI and Performance Budget workflows are pre-configured in `.github/workflows/`. They need `PAYLOAD_URL` as a GitHub Actions secret to build the frontend during checks.
- For detailed commands, see the Railway guide (`railway-payload-deploy.md`) and Cloudflare Pages guide (`cloudflare-pages-deploy.md`).

### Getting Started Guide

After the setup wizard completes, walk the user through getting the project running locally. Use the project's detected package manager (replace `[pm]` with bun/npm/pnpm/yarn).

#### Step 1: Reset Environment Files

The template comes with `.env` and `.env.production` files pre-filled with the template project's values. The wizard detects this and resets them for the new project.

**Claude does this automatically:**
1. Generate a new `PAYLOAD_SECRET` via `openssl rand -hex 32`
2. Keep `PAYLOAD_ADMIN_EMAILS` and `PAYLOAD_ADMIN_PASSWORD` (from auto-memory)
3. Clear all template-specific values: `DATABASE_URI`, R2 credentials, `CLOUDFLARE_DEPLOY_HOOK_URL`
4. Set localhost defaults: `PAYLOAD_PUBLIC_SERVER_URL=http://localhost:3000`, `CLIENT_URI=http://localhost:4321`
5. In `.env.production`: clear all values except admin accounts, add comments showing where to find each value

Then present the guided checklist (see Template Environment Reset section) to walk the user through creating the required services one at a time.

#### Step 2: Set Up R2 Media Storage

**Do this BEFORE starting dev servers.** Without R2 credentials, the s3Storage plugin is disabled and Payload falls back to local disk storage. Files uploaded locally won't exist in production, and re-uploading everything to R2 later is tedious. Set up R2 early so all media goes there from the start - even during local development.

Walk the user through:
1. Create R2 bucket in Cloudflare dashboard, enable public access
2. Create R2 API token with Object Read & Write permissions
3. Add all R2 values to `backend/.env`: `R2_BUCKET`, `R2_ENDPOINT`, `R2_PUBLIC_URL`, `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`

#### Step 3: Install Dependencies

The template's `node_modules` are stale from the copy - always remove them before installing fresh.

```
cd backend && rm -rf node_modules && [pm] install
cd frontend && rm -rf node_modules && [pm] install
```

Or if the root has a workspace setup, `rm -rf node_modules && [pm] install` from the root.

#### Step 3.5: Run Database Migrations (MANDATORY before starting servers)

**This step MUST happen before starting dev servers.** The fresh database has no tables yet. Starting the backend without running migrations first will cause errors because Payload queries tables that don't exist.

```
cd backend && pnpm payload migrate
```

Expected output: all migrations run sequentially with `INFO: Migrated: ...` for each one, ending with `INFO: Done.`

**If DATABASE_URI is not set:** Skip this step and prompt the user to set up the database first (Step 1 in the guided checklist). Migrations cannot run without a database connection.

**If migrations fail:** Check that `DATABASE_URI` in `backend/.env` points to the correct database and that the database is reachable.

#### Step 4: Start Both Dev Servers

Two terminal windows needed:

```
Terminal 1 (backend):  cd backend && pnpm run dev    -> Payload admin on localhost:3000
Terminal 2 (frontend): cd frontend && bun run dev    -> Astro on localhost:4321
```

**Important:** The backend uses `pnpm run dev` (not bun) because Payload requires Node.js as its runtime. Bun works as a package manager for installing dependencies, but the dev server must run through pnpm/Node.js. The frontend (Astro) has full Bun support.

Or from the root folder using convenience scripts: `bun run dev:backend` and `bun run dev:frontend` (these delegate to the correct package managers).

#### Step 5: Log Into Admin Panel

Open `localhost:3000/admin` in the browser.

**If admin seeding is configured** (dev-context.md has `## Admin Seeding`): Admin accounts were automatically created on first startup using the emails from `PAYLOAD_ADMIN_EMAILS`. Log in with any of those emails and the password from `PAYLOAD_ADMIN_PASSWORD` (default: `changeme123`).

**If no admin seeding:** Payload will show a registration form to create the first admin user.

#### Step 6: Start Adding Content

Once logged in, the sidebar shows all collections (Posts, Pages, Categories, Tags, Media). Start creating categories and tags, then write posts and pages.

**Communication tip:** Walk the user through these steps one at a time after setup completes. Don't dump all steps at once - ask if they're ready for the next step after each one completes.
