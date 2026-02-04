# [Project Name] - What You're Building

> **For humans:** This is the detailed document where you plan everything about your app.
> **For Claude:** There's a shorter version at `.claude/projectbrief.md` that Claude reads when building.

**Version:** 1.0
**Last Updated:** [Date]
**Your Name:** [Name]
**Status:** Planning | Building | Live

---

## The Big Picture

**What is it?** [In 2-3 sentences, describe what you're building and why it's cool]

**When do you want it done?** [Your target date or timeframe]

**Who's it for?** [Who will use this]

**What problem does it solve?** [The main thing this helps with]

---

## 1. Your Goals

### What You Want to Achieve

- **Goal 1:** [What you want to accomplish with this project]
- **Goal 2:** [How you want to make money or grow]
- **Goal 3:** [How you want to stand out]

### What Your Users Want

- **Goal 1:** [What your users want to do]
- **Goal 2:** [Problems your users need solved]
- **Goal 3:** [How your app makes their life better]

### How You'll Know It's Working

| What to Track | Your Target | When to Check | How to Measure |
|---------------|-------------|---------------|----------------|
| [Thing to measure] | [Your goal number] | [When] | [How you'll track it] |
| [Example: New users] | [1,000 people] | [3 months after launch] | [Analytics dashboard] |
| [Example: People coming back] | [40% each month] | [6 months] | [Check who returns] |

---

## 2. Who's It For?

### Main User: [Give them a name, like "Sarah the Freelancer"]

**About them:**
- Age: [Age range]
- Where they live: [Location]
- What they do: [Job or role]
- Tech comfort: [Beginner / Comfortable / Expert]

**What they want:**
- [What they're trying to do]
- [Why they need your app]
- [What "success" looks like for them]

**Their frustrations:**
- [What annoys them now]
- [Problems with current solutions]
- [What's missing]

**How they currently handle it:**
- [How they solve this problem today]
- [Tools they already use]
- [How they make decisions]

### Other Users: [Name another type, like "Mike the Small Business Owner"]

[Repeat the same structure for other types of users]

---

## 3. What Users Can Do

### Main Feature 1: [Feature Name]

**Users can:** [What they can do]
**Why it matters:** [Why they want this]

**What counts as "done":**
- [ ] [Something specific they can do]
- [ ] [Something you can test]
- [ ] [Handles weird edge cases]

### Main Feature 2: [Feature Name]

[Same structure]

### Other Things Users Want to Do

- **Signing Up & Logging In**
  - Sign up quickly and easily
  - Log in securely to protect their stuff
  - Log in with Google/GitHub (easier than passwords)

- **Main Features**
  - [List 5-10 things users want to do with your app]

- **Managing Their Data**
  - Create new items
  - View their items
  - Edit items
  - Delete items they don't want

- **Staying Updated**
  - Get notified about important stuff
  - Receive emails when needed
  - See activity in the app

---

## 4. Features You Need

### 4.1 Signing Up & Logging In

**Must have for launch:**
- Email and password login
- Google/GitHub login (easier for users)
- "Forgot password" feature
- Email verification
- Stay logged in between sessions

**Nice to have later:**
- Extra security (two-factor authentication)
- Business login options
- Passwordless login (more secure)

### 4.2 [Your Main Feature 1]

**Must have:**
- [What this feature does]
- [What users can input and get back]
- [How it works / the rules]
- [Who can do what]

**Nice to have:**
- [Cool extras you can add later]
- [Advanced stuff]

### 4.3 [Your Main Feature 2]

[Same structure for each big feature]

### 4.4 Storing Data

**What you need:**
- [What information gets saved]
- [How different data connects]
- [How long to keep data]
- [Backup plan if something goes wrong]

### 4.5 Connecting to Other Services

**Need for launch:**
- [Service name] - [What it does for your app]
- [Service name] - [What it does for your app]

**Maybe later:**
- [Other services you might add]

---

## 5. How It Should Work

### Speed

- Pages load in under 2 seconds
- App responds quickly (under half a second)
- Interactive in under 3 seconds
- Can handle [X] people using it at once

### Security (Keeping Data Safe)

- Data is encrypted when transmitted
- Data is encrypted when stored
- Follows privacy laws (GDPR, etc.)
- Regular security checks
- Validates all user input
- Protected against hacking attempts

### Growth (Handling More Users)

- Can support [X] users in the first year
- Database can grow as needed
- Fast loading from anywhere in the world
- Can add more servers when needed

### Accessibility (Works for Everyone)

- Usable by people with disabilities
- Works with keyboard (no mouse needed)
- Works with screen readers
- Easy to read colors and text
- Works on phones, tablets, and computers

### Where It Works

- Modern browsers (Chrome, Firefox, Safari, Edge)
- Mobile browsers (iPhone, Android)
- Gracefully falls back on older browsers

---

## 6. User Experience & Design

### UX Flow

**Onboarding Flow:**
```
Landing Page → Sign Up → Email Verification → Profile Setup → Dashboard
```

**Core User Journey:**
```
[Map out the primary user flow through the application]
Login → [Action 1] → [Action 2] → [Result/Success State]
```

**Key Screens/Pages:**

1. **Landing Page**
   - Purpose: [Convert visitors to signups]
   - Key elements: [Hero section, features, pricing, CTA]

2. **Dashboard**
   - Purpose: [Main hub for user activities]
   - Key elements: [Overview, quick actions, recent items]

3. **[Feature Page 1]**
   - Purpose: [Specific functionality]
   - Key elements: [Forms, data tables, actions]

[Continue for all major screens]

### Design Principles

- **Simplicity:** Minimize cognitive load, clear hierarchy
- **Consistency:** Reusable components, predictable patterns
- **Feedback:** Loading states, error messages, success confirmations
- **Accessibility:** Inclusive design for all users

### UI Components Needed

- [ ] Navigation (header, sidebar, breadcrumbs)
- [ ] Forms (inputs, selects, checkboxes, validation)
- [ ] Data display (tables, cards, lists)
- [ ] Modals/dialogs
- [ ] Notifications/toasts
- [ ] Loading states
- [ ] Empty states
- [ ] Error states

---

## 7. Content Strategy

### Messaging & Tone

- **Brand Voice:** [Professional, friendly, technical, casual, etc.]
- **Tone:** [Encouraging, instructive, conversational, etc.]

### Content Requirements

- **Marketing Copy:** Landing page, feature descriptions, pricing
- **In-App Copy:** Onboarding, tooltips, help text, error messages
- **Documentation:** User guides, API docs, FAQs
- **Email Templates:** Welcome, verification, notifications, marketing

---

## 8. Technical Architecture (High-Level)

### Tech Stack Overview

**Frontend:**
- Framework: [Next.js, React, Vue, etc.]
- Styling: [Tailwind, CSS-in-JS, etc.]
- Component library: [Shadcn UI, Material UI, etc.]

**Backend:**
- API: [REST, GraphQL, tRPC, etc.]
- Database: [PostgreSQL, MongoDB, etc.]
- ORM: [Prisma, Drizzle, etc.]

**Infrastructure:**
- Hosting: [Vercel, AWS, etc.]
- Database hosting: [Neon, Supabase, etc.]
- CDN: [Cloudflare, Vercel Edge, etc.]

**Third-Party Services:**
- Authentication: [Better Auth, Clerk, etc.]
- Payments: [Stripe, Paddle, etc.]
- Email: [Resend, SendGrid, etc.]
- Analytics: [PostHog, Plausible, etc.]

### Database Schema (Conceptual)

**Users Table:**
```
users
- id (uuid, primary key)
- email (string, unique)
- name (string)
- created_at (timestamp)
- updated_at (timestamp)
```

**[Main Entity Table]:**
```
[entity_name]
- id (uuid, primary key)
- user_id (uuid, foreign key → users)
- [field_name] (type)
- created_at (timestamp)
- updated_at (timestamp)
```

[List all major database entities and their key relationships]

### API Endpoints (Planned)

**Authentication:**
- `POST /api/auth/signup` - Create new account
- `POST /api/auth/login` - Authenticate user
- `POST /api/auth/logout` - End session
- `POST /api/auth/reset-password` - Password reset

**[Feature 1]:**
- `GET /api/[resource]` - List all
- `POST /api/[resource]` - Create new
- `GET /api/[resource]/:id` - Get single
- `PUT /api/[resource]/:id` - Update
- `DELETE /api/[resource]/:id` - Delete

[Continue for all major features]

---

## 9. Release Plan & Milestones

### Phase 1: MVP (Weeks 1-4)

**Goal:** Launch core functionality to initial users

**Features:**
- [ ] User authentication (email/password)
- [ ] Basic dashboard
- [ ] [Core feature 1] - basic version
- [ ] [Core feature 2] - basic version
- [ ] Responsive design

**Success Criteria:**
- 50 beta users
- Core workflows functional
- No critical bugs

### Phase 2: Beta (Weeks 5-8)

**Goal:** Refine based on feedback, add key features

**Features:**
- [ ] Social login
- [ ] Enhanced [feature 1]
- [ ] [New feature 3]
- [ ] Email notifications
- [ ] User settings

**Success Criteria:**
- 200+ active users
- Positive user feedback
- Key metrics trending upward

### Phase 3: Public Launch (Weeks 9-12)

**Goal:** Open to public, marketing push

**Features:**
- [ ] All MVP features polished
- [ ] Payment integration
- [ ] Public marketing site
- [ ] Documentation
- [ ] Analytics dashboard

**Success Criteria:**
- 1,000+ signups in first month
- Target conversion rates met
- Infrastructure stable

### Future Phases

**Phase 4: Growth (3-6 months)**
- Advanced features
- Integrations
- Mobile apps
- API access

**Phase 5: Scale (6-12 months)**
- Enterprise features
- Advanced analytics
- White-label options
- International expansion

---

## 10. Risks & Mitigation

### Technical Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| [Database scaling issues] | High | Medium | [Start with scalable architecture, monitoring] |
| [Third-party API downtime] | Medium | Low | [Implement fallbacks, error handling] |
| [Security vulnerabilities] | High | Low | [Security audits, best practices, pen testing] |

### Business Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| [Low user adoption] | High | Medium | [Beta testing, user research, marketing] |
| [Competitive threats] | Medium | Medium | [Unique features, better UX, faster iteration] |
| [Budget overruns] | Medium | Low | [Phased approach, MVP focus, cost monitoring] |

### User Experience Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| [Complex onboarding] | High | Medium | [User testing, guided tours, simple flows] |
| [Poor performance] | High | Low | [Performance testing, optimization, monitoring] |

---

## 11. Open Questions & Decisions Needed

**Product Questions:**
- [ ] Should we support multi-tenancy from day 1?
- [ ] What's the pricing model? (Freemium, subscription, usage-based?)
- [ ] Do we need mobile apps or is responsive web sufficient?

**Technical Questions:**
- [ ] Which database: PostgreSQL or something else?
- [ ] Self-hosted auth or third-party service?
- [ ] Which deployment platform: Vercel, AWS, or hybrid?

**Business Questions:**
- [ ] What's the go-to-market strategy?
- [ ] Who are our initial beta users?
- [ ] What's the support model?

---

## 12. Appendix

### Competitive Analysis

**Competitor 1: [Name]**
- Strengths: [What they do well]
- Weaknesses: [Where they fall short]
- Differentiation: [How we're different/better]

**Competitor 2: [Name]**
[Repeat structure]

### User Research Summary

[Link to or summarize any user interviews, surveys, or research conducted]

### Design Mockups

[Links to Figma, screenshots, or design files]

### Related Documents

- [Link to Technical Architecture Document]
- [Link to API Specification]
- [Link to Marketing Plan]
- [Link to User Research Report]
- **AI Implementation Context**: `/.claude/projectbrief.md` (concise technical brief for AI code generation)

---

## 13. Implementation Plan

This high-level plan outlines the major implementation steps. Detailed tasks are tracked in `.claude/tasks.md` for day-to-day development.

### Phase 1: Foundation (Weeks 1-2)

**Goal:** Set up core infrastructure and authentication

1. **Initialize project and configure development environment**
   - Set up [Framework] project with TypeScript
   - Configure [Database Provider] and [ORM]
   - Install and configure [UI Component Library]
   - Set up environment variables and secrets management

2. **Implement authentication system**
   - Configure [Auth Provider] with required auth methods
   - Build login, signup, and password reset flows
   - Implement session management
   - Add email verification if required
   - Test auth flows end-to-end

3. **Create base application layout**
   - Build main navigation (header/sidebar)
   - Create page layouts (authenticated vs public)
   - Implement responsive design foundation
   - Add dark mode support (if required)

4. **Set up database schema**
   - Design and document core entities and relationships
   - Create initial migration files
   - Set up database seeding for development
   - Test database connections and queries

---

### Phase 2: Core Features (Weeks 3-5)

**Goal:** Build MVP features

5. **Implement [Core Feature 1]**
   - Create database models and migrations
   - Build API endpoints (CRUD operations)
   - Develop UI components and forms
   - Add validation and error handling
   - Write initial tests

6. **Implement [Core Feature 2]**
   - [Follow same structure as above]

7. **Implement [Core Feature 3]**
   - [Follow same structure as above]

8. **Add data validation and error handling**
   - Implement Zod schemas for all inputs
   - Create error boundary components
   - Add loading states throughout app
   - Implement toast notifications
   - Handle edge cases and empty states

---

### Phase 3: Integration & Enhancement (Weeks 6-7)

**Goal:** Add third-party integrations and advanced features

9. **Integrate [Third-Party Service 1]** (if applicable)
   - Example: Set up Stripe for payments
   - Configure API keys and webhooks
   - Implement integration logic
   - Test integration flows
   - Handle errors and edge cases

10. **Integrate [Third-Party Service 2]** (if applicable)
    - Example: Connect to AI API for content generation
    - Build prompt engineering and response handling
    - Add streaming support (if needed)
    - Implement rate limiting and error handling

11. **Implement background jobs and workflows** (if applicable)
    - Set up [Workflow/Job Queue System]
    - Create scheduled tasks
    - Implement email notifications
    - Add retry logic and monitoring

12. **Add real-time features** (if applicable)
    - Configure real-time subscriptions
    - Implement live updates in UI
    - Handle connection states
    - Test synchronization logic

---

### Phase 4: Polish & Optimization (Week 8)

**Goal:** Refine UX, optimize performance, prepare for launch

13. **UI/UX polish and accessibility**
    - Refine component styling and interactions
    - Ensure WCAG 2.1 Level AA compliance
    - Test keyboard navigation
    - Add proper ARIA labels
    - Test with screen readers

14. **Performance optimization**
    - Implement code splitting and lazy loading
    - Optimize images and assets
    - Add caching strategies
    - Minimize bundle size
    - Test Core Web Vitals

15. **Testing and quality assurance**
    - Write integration tests for critical flows
    - Perform manual testing across browsers/devices
    - Test error scenarios
    - Security review and penetration testing
    - Load testing (if applicable)

16. **Documentation and deployment prep**
    - Write user documentation/help articles
    - Create API documentation
    - Set up production environment
    - Configure monitoring and analytics
    - Prepare deployment pipeline

---

### Phase 5: Launch (Week 9+)

**Goal:** Deploy to production and monitor

17. **Deploy to production**
    - Deploy application to [Deployment Platform]
    - Configure domain and SSL
    - Set up CDN (if applicable)
    - Enable monitoring and error tracking
    - Test production environment

18. **Post-launch monitoring and iteration**
    - Monitor analytics and user behavior
    - Track error rates and performance
    - Gather user feedback
    - Plan iteration roadmap
    - Address critical issues

---

### Implementation Task Tracking

**Detailed tasks are managed in:** `/.claude/tasks.md`

The tasks.md file contains:
- ✅ Completed tasks with timestamps
- 🚧 In-progress tasks with current status
- 📋 Pending tasks with priority levels
- 🐛 Bugs and issues to fix
- 💡 Future enhancements and ideas

**This high-level plan provides the strategic view, while tasks.md tracks daily execution.**

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | [Date] | [Name] | Initial draft |
| 1.1 | [Date] | [Name] | [Description of changes] |
