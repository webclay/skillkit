# Existing Codebase Detection

## Reference for Claude

When setting up a project that already has code, follow this detection process.

## What to Scan

### 1. Package.json Dependencies

Look for these patterns:

| Dependency | Indicates |
|------------|-----------|
| `next` | Next.js framework |
| `@tanstack/start` | TanStack Start framework |
| `react-native` | Mobile app |
| `expo` | Expo mobile framework |
| `prisma` or `@prisma/client` | Prisma ORM |
| `drizzle-orm` | Drizzle ORM |
| `@supabase/supabase-js` | Supabase |
| `convex` | Convex backend |
| `better-auth` | Better Auth |
| `next-auth` | NextAuth.js |
| `@clerk/nextjs` | Clerk auth |
| `stripe` | Stripe payments |
| `@polar-sh/sdk` | Polar payments |
| `resend` | Resend email |
| `ai` or `@ai-sdk/*` | Vercel AI SDK |
| `openai` | OpenAI SDK |
| `tailwindcss` | Tailwind CSS |
| `@radix-ui/*` | Radix UI (Shadcn base) |

### 2. File Structure

| Files/Folders | Indicates |
|---------------|-----------|
| `app/` directory | Next.js App Router |
| `pages/` directory | Next.js Pages Router |
| `src/routes/` | TanStack Start |
| `prisma/schema.prisma` | Prisma schema |
| `drizzle/` folder | Drizzle config |
| `convex/` folder | Convex functions |
| `supabase/` folder | Supabase config |
| `components/ui/` | Likely Shadcn UI |
| `ios/` and `android/` | React Native |

### 3. Config Files

| File | Indicates |
|------|-----------|
| `next.config.js/ts` | Next.js |
| `tailwind.config.js/ts` | Tailwind CSS |
| `tsconfig.json` | TypeScript |
| `.env` or `.env.local` | Environment variables |
| `convex.json` | Convex |
| `wrangler.toml` | Cloudflare |
| `vercel.json` | Vercel deployment |
| `netlify.toml` | Netlify deployment |

## Detection Output

After scanning, present findings:

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

## Clarifying Questions

After detection, ask:

1. **Confirm accuracy:**
   ```
   Does this look right? Anything I missed or got wrong?
   ```

2. **Fill gaps:**
   ```
   I couldn't detect what you're using for [X]. What are you using?
   ```

3. **Project purpose:**
   ```
   I can see the code structure, but could you describe what this project does?
   This helps me understand the context better.
   ```

## Creating Documentation from Detection

Use detected info to populate:

### project/projectbrief.md

```markdown
# [Project Name]

## Overview
[Ask user to describe]

## Tech Stack
- **Framework:** [detected]
- **Database:** [detected]
- **Authentication:** [detected]
- **UI:** [detected]
- **Deployment:** [detected]

## Features
[Infer from code + ask user to confirm]

## Database Schema
[Extract from Prisma schema if exists]
```

### project/tasks.md

For existing codebases, focus on:
- Features that appear incomplete
- Missing tests
- Documentation gaps
- Optimization opportunities

Ask user what they want to work on next.

## Handling Incomplete Detection

If uncertain about something:

```
I'm not 100% sure about your [authentication/database/etc.].

I see [what I found], but it could also be [alternative].

Which one is it?
```

Never guess - always confirm with the user.
