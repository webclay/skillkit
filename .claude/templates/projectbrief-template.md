# [Project Name] - Project Brief

> **Purpose:** This document provides Claude with essential context for code generation and implementation.
> **For humans:** See `/docs/prd.md` for the full Product Requirements Document.
>
> **Living Document:** This brief is filled in progressively as decisions are made with the user.
>
> **No Assumptions Rule:** Only fill sections where the user provided explicit input.
> For anything not discussed, use `[TBD - discuss before implementation]`.
> When starting a task that involves a TBD section, stop and ask the user first.
> This applies to the entire document - tech stack, features, schema, endpoints, UI, everything.

---

## Project Overview

**What we're building:** [1-2 sentence description of the product]

**Primary users:** [Who uses this]

**Core value:** [Main problem it solves]

**Target launch:** [Timeframe]

---

## Tech Stack

### Framework & Meta-framework
- **Primary:** [Next.js / TanStack Start / React / etc.]
- **Routing:** [App Router / Pages Router / File-based / etc.]
- **Rendering:** [SSR / SSG / CSR / Hybrid]

### Database & ORM
- **Database:** [PostgreSQL / MongoDB / etc.]
- **Hosting:** [Neon / Supabase / Local / etc.]
- **ORM/Client:** [Prisma / Drizzle / Convex / Supabase Client / etc.]
- **Real-time:** [Yes/No - if yes, specify: Convex / Supabase Realtime / WebSockets]

### Authentication
- **Provider:** [Better Auth / Clerk / NextAuth / Custom]
- **Methods:** [Email/password, Social (Google, GitHub), Magic links, etc.]
- **Session:** [Cookie-based / JWT / Database sessions]

### UI & Styling
- **Component Library:** [Shadcn UI / Untitled UI / Custom]
- **Styling:** [Tailwind CSS / CSS Modules / Styled Components]
- **Icons:** [Lucide / Heroicons / etc.]
- **Theme:** [Dark mode support: Yes/No]

### Backend & API
- **API Pattern:** [REST / GraphQL / tRPC / Server Actions]
- **API Routes:** [Next.js API routes / Separate backend / Serverless functions]
- **Validation:** [Zod / Yup / etc.]

### AI Integration (if applicable)
- **AI Provider:** [OpenAI / Anthropic / OpenRouter / Multiple]
- **SDK:** [Vercel AI SDK / Direct API / LangChain]
- **Features:** [Text generation / Streaming / RAG / Function calling / etc.]

### Additional Services
- **Payments:** [Stripe / Paddle / None]
- **Email:** [Resend / SendGrid / None]
- **File Storage:** [Supabase Storage / S3 / Cloudinary / etc.]
- **Analytics:** [PostHog / Vercel Analytics / Plausible / None]
- **Workflows:** [Workflow DevKit / Motia / None]
- **Web Scraping:** [Firecrawl / None]

### Deployment
- **Platform:** [Vercel / Netlify / AWS / Self-hosted]
- **Environment:** [Serverless / Edge / Traditional server]

---

## Core Features (MVP)

### Feature 1: [Name]
**What:** [Brief description of what this feature does]

**User flow:**
```
[Step 1] → [Step 2] → [Step 3] → [Success state]
```

**Data requirements:**
- [Key data entities and relationships]

**Implementation notes:**
- [Specific technical considerations]
- [Edge cases to handle]

---

### Feature 2: [Name]
[Repeat structure]

---

### Feature 3: [Name]
[Repeat structure]

---

## Database Schema (Simplified)

### Core Entities

```typescript
// Users
users {
  id: uuid
  email: string
  name: string
  created_at: timestamp
}

// [Main Entity 1]
[entity_name] {
  id: uuid
  user_id: uuid (FK → users)
  [field]: type
  created_at: timestamp
  updated_at: timestamp
}

// [Main Entity 2]
[entity_name] {
  // ...
}

// Relationships:
// - [Describe key relationships between entities]
```

---

## API Endpoints (Planned)

### Authentication
- `POST /api/auth/signup`
- `POST /api/auth/login`
- `POST /api/auth/logout`

### [Resource 1]
- `GET /api/[resource]` - List
- `POST /api/[resource]` - Create
- `GET /api/[resource]/[id]` - Get
- `PUT /api/[resource]/[id]` - Update
- `DELETE /api/[resource]/[id]` - Delete

[List key API endpoints]

---

## Key User Stories (Implementation Focus)

### Authentication & Onboarding
- [ ] User can sign up with email/password
- [ ] User can log in with Google
- [ ] User receives verification email
- [ ] User can reset password

### [Feature Area 1]
- [ ] [Specific implementable user story]
- [ ] [Acceptance criteria in technical terms]

### [Feature Area 2]
- [ ] [User story]

[List 10-15 key user stories that map directly to implementation tasks]

---

## UI Components Needed

**Layout:**
- [ ] Main navigation (header/sidebar)
- [ ] Footer
- [ ] Page layouts (dashboard, auth, public)

**Forms:**
- [ ] Login form
- [ ] Signup form
- [ ] [Feature-specific form 1]
- [ ] [Feature-specific form 2]

**Data Display:**
- [ ] Data table with sorting/filtering
- [ ] Cards/list views
- [ ] [Feature-specific display]

**Interactive:**
- [ ] Modals/dialogs
- [ ] Toast notifications
- [ ] Loading states
- [ ] Error boundaries

**Feature-Specific:**
- [ ] [Custom component 1]
- [ ] [Custom component 2]

---

## Implementation Priorities

### Phase 1: Foundation (Week 1)
**Goal:** Set up project infrastructure and authentication

- [ ] Initialize Next.js project with TypeScript
- [ ] Set up database (create Neon project, configure Prisma)
- [ ] Install and configure Shadcn UI
- [ ] Implement Better Auth (email/password + Google)
- [ ] Create base layout and navigation
- [ ] Set up environment variables

### Phase 2: Core Features (Weeks 2-3)
**Goal:** Build MVP features

- [ ] Implement [Feature 1]
- [ ] Implement [Feature 2]
- [ ] Implement [Feature 3]
- [ ] Add data validation (Zod schemas)
- [ ] Error handling and loading states

### Phase 3: Polish & Launch Prep (Week 4)
**Goal:** Refine UX and prepare for launch

- [ ] UI/UX polish
- [ ] Responsive design testing
- [ ] Performance optimization
- [ ] Error messages and empty states
- [ ] Deploy to production

---

## Code Generation Guidelines

### File Structure Preferences
```
app/
├── (auth)/
│   ├── login/
│   ├── signup/
│   └── reset-password/
├── (dashboard)/
│   ├── layout.tsx
│   └── [feature]/
├── api/
│   └── [endpoints]/
└── layout.tsx

components/
├── ui/           # Shadcn components
├── auth/         # Auth-related
└── [feature]/    # Feature-specific

lib/
├── db.ts         # Database client
├── auth.ts       # Auth config
├── utils.ts      # Utilities
└── validations/  # Zod schemas
```

### Coding Standards
- **TypeScript:** Use explicit types for function params and returns
- **Error Handling:** Try-catch blocks with proper error messages
- **Validation:** Zod schemas for all user inputs
- **Async:** Use async/await, not promise chains
- **Components:** Server Components by default, Client only when needed
- **Imports:** Named imports, avoid barrel files
- **Formatting:** Ultracite/Biome for linting and formatting

### Naming Conventions
- **Files:** kebab-case for files (user-profile.tsx)
- **Components:** PascalCase (UserProfile)
- **Functions:** camelCase (getUserProfile)
- **Constants:** UPPER_SNAKE_CASE (API_URL)
- **Types/Interfaces:** PascalCase (UserProfile)

### Security Requirements
- Input validation on all user inputs
- SQL injection protection (via ORM)
- XSS protection (proper escaping)
- CSRF protection (built into framework)
- Secure session handling
- Environment variables for secrets (never hardcode)

---

## Environment Variables Required

```bash
# Database
DATABASE_URL="postgresql://..."
DIRECT_URL="postgresql://..."  # For migrations (Neon)

# Authentication
BETTER_AUTH_SECRET="..."
BETTER_AUTH_URL="http://localhost:3000"  # Production: https://yourdomain.com

# Google OAuth (if using)
GOOGLE_CLIENT_ID="..."
GOOGLE_CLIENT_SECRET="..."

# Email (if using)
RESEND_API_KEY="..."

# AI (if using)
OPENAI_API_KEY="..."  # or ANTHROPIC_API_KEY, etc.

# Payments (if using)
STRIPE_SECRET_KEY="..."
STRIPE_PUBLISHABLE_KEY="..."
STRIPE_WEBHOOK_SECRET="..."

# Analytics (if using)
NEXT_PUBLIC_POSTHOG_KEY="..."
```

---

## Common Patterns & Utilities

### Database Queries
```typescript
// Use Prisma client
import { prisma } from '@/lib/db'

// Example query pattern
const items = await prisma.item.findMany({
  where: { userId },
  include: { user: true },
  orderBy: { createdAt: 'desc' }
})
```

### Authentication Checks
```typescript
// Server component
import { auth } from '@/lib/auth'

const session = await auth()
if (!session) redirect('/login')
```

### Form Validation
```typescript
// Use Zod
import { z } from 'zod'

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8)
})
```

### API Error Handling
```typescript
try {
  // Logic
} catch (error) {
  console.error('Error:', error)
  return NextResponse.json(
    { error: 'Something went wrong' },
    { status: 500 }
  )
}
```

---

## Feature Dependencies

> **Purpose:** This map helps Claude understand how features connect. When working on one feature, Claude will check this map and warn you if changes might impact other areas.

### Dependency Map

- **Database Schema** → All features (foundational - changes here affect everything)
- **Authentication** → Dashboard, User Settings, Protected Routes, Payments
- **UI Components** → All pages and features
- **[Feature 1]** → [Features that depend on it]
- **[Feature 2]** → [Features that depend on it]
- **Payments** → User Settings, Subscription Pages, Access Control

### Notes
- Update this map when adding new features
- Claude will reference this before making changes
- Helps prevent breaking changes across features

---

## Known Constraints & Considerations

### Performance
- Target page load: < 2 seconds
- API response: < 500ms
- Image optimization: Use Next.js Image component

### Browser Support
- Modern browsers (Chrome, Firefox, Safari, Edge - last 2 versions)
- Mobile responsive required

### Accessibility
- WCAG 2.1 Level AA (Shadcn UI handles most of this)
- Keyboard navigation
- Screen reader support

### Data Limits
- [Any constraints on data volume, file sizes, etc.]

---

## Questions & Assumptions

### Assumptions
- User has basic knowledge of [framework]
- Development on local machine first, deploy to [platform]
- Using free tiers initially for third-party services

### Open Questions
- [ ] [Any unresolved technical decisions]
- [ ] [Features that need clarification]

### TBD Sections
> List any sections marked as TBD that need discussion before implementation.
> When starting work that touches these areas, ask the user first.

- [ ] [Section name] - [Brief note on what needs to be decided]

---

## Quick Reference Links

**Documentation:**
- Tech stack skills: `.claude/skills/`
- Full PRD: `/docs/prd.md`

**External Resources:**
- [Framework docs]
- [Database docs]
- [Auth provider docs]

---

**Last updated:** [Auto-generated by setup command]
**Stack modules loaded:** [List of stack .md files that were included in .claude/]
