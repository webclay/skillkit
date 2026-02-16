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

Environment variables depend on the deployment choice.

Environment variables depend on the database and deployment choices.

#### PostgreSQL Setup

**`.env` (local development):**
```
# App
APP_URL=http://localhost:4321

# Payload CMS
PAYLOAD_URL=http://localhost:3000
PAYLOAD_SECRET=dev-secret-change-in-production
DATABASE_URL=postgres://localhost:5432/payload-dev
```

**`.env.production`:**
```
# App
APP_URL=  # Your domain (e.g., https://example.com)

# Payload CMS
PAYLOAD_URL=  # Your Payload deployment URL
PAYLOAD_SECRET=  # Generate: openssl rand -base64 32
DATABASE_URL=  # From your database provider (Postgres connection string)
```

**Database hosting by backend platform:**

| Backend + DB Provider | DATABASE_URL source |
|----------------------|---------------------|
| Railway + Railway Postgres | Auto-provided by Railway |
| Cloudflare Workers + Supabase | From Supabase dashboard -> Settings -> Database |
| Cloudflare Workers + Neon | From Neon dashboard |
| Vercel + Supabase | From Supabase dashboard |
| Render + Render Postgres | Auto-provided by Render |

#### SQLite / D1 Setup

**`.env` (local development):**
```
# App
APP_URL=http://localhost:4321

# Payload CMS
PAYLOAD_URL=http://localhost:3000
PAYLOAD_SECRET=dev-secret-change-in-production
```

Locally, Payload uses a SQLite file - no DATABASE_URL needed.

**`.env.production` (Cloudflare D1):**
```
# App
APP_URL=  # Your domain (e.g., https://example.com)

# Payload CMS
PAYLOAD_URL=  # Your Workers URL (e.g., https://cms.example.com)
PAYLOAD_SECRET=  # Generate: openssl rand -base64 32
```

D1 is configured via `wrangler.toml` bindings, not environment variables.

#### MongoDB Setup

**`.env` (local development):**
```
# App
APP_URL=http://localhost:4321

# Payload CMS
PAYLOAD_URL=http://localhost:3000
PAYLOAD_SECRET=dev-secret-change-in-production
MONGODB_URI=mongodb://localhost:27017/payload-dev
```

**`.env.production`:**
```
# App
APP_URL=  # Your domain (e.g., https://example.com)

# Payload CMS
PAYLOAD_URL=  # Your Payload deployment URL
PAYLOAD_SECRET=  # Generate: openssl rand -base64 32
MONGODB_URI=  # From MongoDB Atlas (connection string)
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

Generate these initial tasks in `project/tasks.md`. Since the template is already in place, tasks focus on getting running, customization, and deployment:

```markdown
## Tasks

### Getting Started
- [ ] Set up environment files (copy .env.example to .env in both backend and frontend)
- [ ] Start the database (Docker Compose or existing Postgres)
- [ ] Install dependencies in both backend and frontend
- [ ] Start both dev servers (backend on :3000, frontend on :4321)
- [ ] Create first admin account at localhost:3000/admin

### Configuration
- [ ] Customize Payload collections for this project's content needs
- [ ] Update Astro pages to match the website's design

### Deployment
- [ ] Deploy Payload to [selected platform]
- [ ] Deploy Astro to [selected platform]
- [ ] Verify CMS-to-frontend connection in production

### Content
- [ ] Create initial content in Payload admin panel
- [ ] Set up navigation structure
- [ ] Add media assets
```

### Getting Started Guide

After the setup wizard completes, walk the user through getting the project running locally. Use the project's detected package manager (replace `[pm]` with bun/npm/pnpm/yarn).

#### Step 1: Set Up Environment Files

Both the backend and frontend have `.env.example` files. Copy them:

```
Backend: Copy backend/.env.example to backend/.env
- Change PAYLOAD_SECRET to a new random string (this secures the admin panel)

Frontend: Copy frontend/.env.example to frontend/.env
- Defaults are fine for local development
```

**Claude should do this for the user** - read the .env.example files, generate a random PAYLOAD_SECRET, and create the .env files.

#### Step 2: Start the Database

**For Cloudflare setup (D1):** Payload uses SQLite locally, no database setup needed. Skip this step.

**For Railway / Render / Vercel setup (Postgres):** The backend includes a Docker Compose file with PostgreSQL pre-configured.

```
cd backend && docker-compose up -d
```

This starts PostgreSQL on port 5432. If the user already has PostgreSQL running locally, they can skip Docker and point `DATABASE_URI` in their `.env` to the existing database.

#### Step 3: Install Dependencies

```
cd backend && [pm] install
cd frontend && [pm] install
```

Or if the root has a workspace setup, just `[pm] install` from the root.

#### Step 4: Start Both Dev Servers

Two terminal windows needed:

```
Terminal 1 (backend):  cd backend && [pm] run dev    -> Payload admin on localhost:3000
Terminal 2 (frontend): cd frontend && [pm] run dev   -> Astro on localhost:4321
```

Or from the root folder using convenience scripts: `[pm] run dev:backend` and `[pm] run dev:frontend`.

#### Step 5: Create Admin Account

Open `localhost:3000/admin` in the browser. Payload will ask to create the first admin user (email + password). This is the login for the content management panel.

#### Step 6: Start Adding Content

Once logged in, the sidebar shows all collections (Posts, Pages, Categories, Tags, Media). Start creating categories and tags, then write posts and pages.

**Communication tip:** Walk the user through these steps one at a time after setup completes. Don't dump all steps at once - ask if they're ready for the next step after each one completes.
