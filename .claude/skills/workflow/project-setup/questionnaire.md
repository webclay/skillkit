# Project Setup Questionnaire

## Reference for Claude

This questionnaire guides new project setup. Follow these rules:

### Formatting Rules

1. **One question at a time** - Never ask multiple questions in one message
2. **Wait for answers** - Don't proceed until user responds
3. **Use plain English** - No technical jargon
4. **Offer recommendations** - Always suggest a best option
5. **Auto-detect project name** - Use the parent folder name

### Question Flow

#### Opening

First, detect the package manager by checking for lock files:
- `bun.lockb` → Bun
- `pnpm-lock.yaml` → pnpm
- `yarn.lock` → Yarn
- `package-lock.json` → npm

If no lock file exists, ask the user.

```
Welcome! I'll help you set up your project.

Project name: **[folder-name]** (from your folder)
Package manager: **[detected or ask]**

Let's get started with a few quick questions.
```

---

#### Question 0: Package Manager (if not detected)

```
Which package manager do you use?

A. bun (Recommended - fastest)
B. pnpm
C. npm
D. yarn

This affects all install commands I'll show you.
```

---

#### Question 1: Project Description

```
In 2-3 sentences, what are you building?

(Example: "An app where freelancers can track their time and send invoices to clients")
```

---

#### Question 2: Target Users

```
Who will use this?

(Example: "Freelancers and small agencies")
```

---

#### Question 3: Application Type

```
What type of application is this?

A. Web application (works in browser)
B. Mobile app (iOS/Android)
C. Both web and mobile
D. Chrome extension
E. WordPress plugin
F. Something else

Just reply with the letter or describe it!
```

---

#### Question 4: Key Features

```
What are the main features you need? (Select all that apply)

- User accounts and login
- Database to store information
- Payments and subscriptions
- Email notifications
- Real-time updates (like chat)
- File uploads
- AI features
- Something else?

Just list what applies, or describe in your own words.
```

---

#### Question 5: Experience Level

```
How much coding experience do you have?

A. None - I describe what I want, Claude builds it
B. Some - I can make small tweaks to code
C. Experienced - I'm comfortable with code

This helps me explain things at the right level.
```

---

#### Question 6: Tech Preferences (Optional)

```
Do you have any technology preferences?

(Example: "I've heard good things about Supabase" or "No preference, recommend something")

If you're not sure, I'll recommend the best options for your project.
```

---

#### Question 7: Timeline

```
How soon do you want to launch?

A. Just exploring / learning
B. Side project, no rush
C. Want to launch within a few months
D. Urgent - need it soon

This helps me prioritize features.
```

---

### After Questionnaire

Present recommendations:

```
Based on your answers, here's what I recommend:

**Project:** [Name]
**Type:** [Web App / Mobile / etc.]

**Recommended Setup:**
- [Framework] - [one-line explanation why]
- [Database] - [one-line explanation why]
- [Auth] - [one-line explanation why]
- [Other relevant tools...]

**Skills I'll install:**
- [skill-name] - for [purpose]
- [skill-name] - for [purpose]

Does this look good? I can explain any choice or swap alternatives.
```

---

### Making Recommendations

**Default Recommended Stack (for most web apps):**

| Layer | Choice | Why |
|-------|--------|-----|
| Framework | TanStack Start | Modern, full-stack React, no vendor lock-in |
| Database | Supabase (Postgres) | Managed, free tier, realtime built-in |
| ORM | Drizzle | Lighter than Prisma, great for serverless |
| Auth | Better Auth | Flexible, works with any DB |
| UI | Tailwind + shadcn/ui | Copy-paste components, easy customization |
| Data Fetching | TanStack Query | Caching, loading states handled |

Based on specific needs, recommend:

| User Needs | Recommend |
|------------|-----------|
| Web app (default) | TanStack Start + Supabase + Drizzle + Better Auth |
| Real-time features | Add Supabase Realtime (built-in) |
| Offline-first app | Electric SQL instead of Supabase Realtime |
| Mobile app | React Native + Expo |
| Chrome extension | Plasmo or WXT |
| AI features | TanStack AI (multi-provider support) |
| Payments | Stripe |
| Nice UI | shadcn/ui (always recommend) |

**Avoid recommending:**
- Next.js (user prefers TanStack Start)
- Prisma for serverless (use Drizzle instead)
- Supabase Auth alone (Better Auth is more flexible)
- Electric SQL with Supabase (use Supabase Realtime instead)

### Handling "I don't know"

If user says they don't know:
1. Acknowledge that's fine
2. Make a recommendation with brief explanation
3. Confirm they're okay with it
4. **Only fill in projectbrief if user explicitly approves**

Example:
```
No problem! For a project like yours, I'd recommend:

**Prisma + Neon** for your database
- Prisma makes database work simple (you describe what you want, it handles the details)
- Neon is a modern database that scales automatically

Sound good?
```

If user doesn't confirm or skips the question, mark as TBD in projectbrief.

### Handling Incomplete Answers

**Never assume.** If the user didn't specify something, don't fill it in.

| User Input | Projectbrief Entry |
|------------|-------------------|
| "I want user accounts" | Auth: `[TBD - methods not specified, discuss before implementation]` |
| "I need a database" | Database: `[TBD - type and provider not specified]` |
| "Some kind of UI" | UI: `[TBD - framework preference not specified]` |
| User approved recommendation | Fill in the approved choice |

### TBD Section Format

When marking sections as TBD, use this format:

```markdown
### Database Schema
[TBD - discuss before implementation]

*Will be defined when we start building features that need data storage.*
```

This tells future Claude sessions to stop and ask before making decisions in this area.

---

## Template Field Mapping

When creating documentation files, map questionnaire answers to template sections:

### project/projectbrief.md Mapping

| Question | Template Section |
|----------|-----------------|
| Q1: What are you building? | `## Project Overview` → "What we're building" |
| Q2: Who will use this? | `## Project Overview` → "Primary users" |
| Q3: Application type | `## Tech Stack` → Framework selection |
| Q4: Key features | `## Core Features (MVP)` |
| Q5: Experience level | (internal - affects explanation depth, not stored) |
| Q6: Tech preferences | `## Tech Stack` → All subsections |
| Q7: Timeline | `## Project Overview` → "Target launch" |

### Additional Fields to Populate

Based on tech recommendations:

| Recommendation | Template Sections to Fill |
|----------------|--------------------------|
| Database choice | `## Tech Stack` → Database & ORM |
| Auth choice | `## Tech Stack` → Authentication |
| UI choice | `## Tech Stack` → UI & Styling |
| Payments | `## Tech Stack` → Additional Services |
| AI features | `## Tech Stack` → AI Integration |

### Sections to Skip by Project Type

| Project Type | Skip These Sections |
|--------------|---------------------|
| Landing page | Database Schema, API Endpoints, AI Integration |
| Simple app | API Endpoints (Planned), Implementation Priorities |
| Full SaaS | None - use full template |

### project/tasks.md Mapping

Initial tasks are generated based on recommended tech stack:

| Tech Stack Component | Initial Tasks |
|---------------------|---------------|
| TanStack Start | "Set up TanStack Start project", "Configure routing" |
| Supabase | "Create Supabase project", "Set up database tables" |
| Drizzle | "Configure Drizzle schema", "Run initial migration" |
| Better Auth | "Configure authentication", "Add login/signup pages" |
| shadcn/ui | "Install UI components", "Set up theme" |
| Stripe | "Configure Stripe", "Add checkout flow" |

### Starter Commands by Package Manager

| Package Manager | Create Command |
|-----------------|----------------|
| bun | `bun create @tanstack/app my-app` |
| pnpm | `pnpm create @tanstack/app my-app` |
| npm | `npx @tanstack/create-app my-app` |
| yarn | `yarn create @tanstack/app my-app` |

### project/changelog.md Mapping

First entry is always the setup session:

```markdown
### [Today's Date] - Project Setup

**Summary:** Initialized project with Claude Code Skills.

**Configured:**
- Tech stack: [Framework], [Database], [Auth]
- Created project/projectbrief.md
- Created initial tasks

**Next:** [First task from project/tasks.md]
```
