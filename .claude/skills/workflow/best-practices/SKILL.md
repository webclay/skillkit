---
name: best-practices
description: Use this skill when following development best practices for task management, commit conventions, or session workflows. Activate when the user mentions best practices, code quality guidelines, commit message standards, or workflow conventions.
---

# Best Practices

Guidelines for effective development workflows including task management, commit messages, and session protocols.

## When to Use This Skill

- User asks about best practices for development
- Creating or reviewing commit messages
- Managing tasks and tracking progress
- Starting or ending a development session
- Discussing code quality standards

## Task Management

### Task Tagging System

Every task in `project/tasks.md` should include status and context:

```markdown
## Task 3: User Authentication
**Status:** in progress
**Priority:** High
**Tags:** #backend #auth #security
**Dependencies:** Task 1, Task 2
```

**Standard Tags:**
- **Domain:** `#frontend`, `#backend`, `#database`, `#api`, `#ui`, `#devops`
- **Type:** `#feature`, `#bug`, `#refactor`, `#docs`, `#test`, `#security`
- **Priority:** `#critical`, `#high`, `#medium`, `#low`

### Automatic Updates

Update `project/tasks.md` when:
- Creating a commit → Mark related tasks as in_progress or complete
- User says "done with [task]" → Update status to complete
- User says "starting [task]" → Update status to in_progress
- Build fails → Add note to relevant task with error details

## Commit Workflow

### Smart Commit Messages

When user says "commit", follow this process:

1. **Analyze changes** - What files were modified?
2. **Match to tasks** - Which task does this relate to?
3. **Update tasks** - Mark checkboxes complete
4. **Generate message** - Use conventional commit format

### Commit Message Format

```
<type>(<scope>): <short description>

<detailed description>

Changes:
- [Bullet point of changes]
- [Another change]

Related: Task X
```

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `refactor` - Code restructuring
- `docs` - Documentation
- `test` - Tests
- `chore` - Maintenance

**Example:**
```
feat(auth): implement login form with validation

Add login form component with email/password fields.
Integrate form validation with Zod.
Add error handling and loading states.

Changes:
- Add LoginForm component
- Add validation schema
- Add auth API route

Related: Task 3
```

## Session Protocols

### Session Start

When starting a new session:

1. **Read** `project/tasks.md` to understand current state
2. **Check** `dev-context.md` for in-progress work
3. **Identify** last worked-on task
4. **Ask user** what they want to work on

**Example greeting:**
```
I see you were working on Task 3 (User Authentication).
Last session you completed the login form.

Would you like to continue with the signup flow, or work on something else?
```

### Session End

When user indicates session is ending:

1. **Review** uncommitted changes
2. **Prompt** to commit if work was done
3. **Update** `project/tasks.md` with progress
4. **Update** `dev-context.md` with current state
5. **Suggest** next steps

**Example wrap-up:**
```
Before you go, let me wrap up:

Uncommitted changes:
- Modified: auth.ts, login.tsx
- Added: validation.ts

Would you like to commit these changes?

I'll update tasks.md with today's progress:
- Completed: Login form component
- In progress: Form validation

Next session suggestion: Add error handling and loading states.
```

## Server-First Rendering (No Flash / No Double-Load)

**The #1 UX problem in modern web apps: content flashes, disappears into a skeleton loader, then reappears.** This happens when data that should be server-rendered is instead fetched on the client after the page loads.

### The Rule

**Load data before rendering. Show a skeleton while loading, then render the page once.** The user should never see content appear, vanish into a loading state, then reappear. The correct flow is:

1. User navigates to a page
2. Skeleton/pending state shows **while the server fetches data**
3. Page renders **once** with complete data
4. Done - no second loading state, no flash

The broken flow is:

1. User navigates to a page
2. Server renders an empty shell (or partial content)
3. Client JS loads, fires `useEffect`, shows a skeleton
4. Data comes back, page re-renders with content

**The difference:** In the good pattern, the skeleton shows **before** the page renders. In the bad pattern, the page renders, then **replaces itself** with a skeleton, then renders again.

### What Causes the Double-Load Flash

1. **Using `useEffect` + `useState` for initial data** - Component renders empty, then fetches, then re-renders
2. **Client-side data fetching for server-available data** - Dashboard data, user profile, page content
3. **Wrapping entire pages in Suspense/loading boundaries** - Shows skeleton for the whole page instead of streaming parts
4. **Hydration mismatch** - Server renders one thing, client JS takes over and re-renders differently
5. **Unnecessary client components** - Making a component `'use client'` when it doesn't need interactivity

### The Fix: Load Data Before Rendering

**TanStack Start - Use loaders with pendingComponent, not useEffect:**

```tsx
// BAD - causes double-load flash
function Dashboard() {
  const [data, setData] = useState(null);
  useEffect(() => {
    fetch('/api/dashboard').then(r => r.json()).then(setData);
  }, []);
  if (!data) return <Skeleton />;  // Shows AFTER empty page renders
  return <DashboardContent data={data} />;
}

// GOOD - skeleton shows while loader runs, page renders once
export const Route = createFileRoute('/dashboard')({
  loader: async () => {
    const data = await fetchDashboardData();
    return { data };
  },
  pendingComponent: () => <DashboardSkeleton />,  // Shows BEFORE page renders
  component: Dashboard,
});

function Dashboard() {
  const { data } = Route.useLoaderData();
  return <DashboardContent data={data} />;  // Renders once with complete data
}
```

**Next.js - Use Server Components with Suspense, not client fetching:**

```tsx
// BAD - client component fetching (double-load)
'use client';
function UserProfile() {
  const [user, setUser] = useState(null);
  useEffect(() => { fetchUser().then(setUser); }, []);
  if (!user) return <Skeleton />;
  return <Profile user={user} />;
}

// GOOD - server component with Suspense boundary for loading
import { Suspense } from 'react';

// Parent page
export default function Page() {
  return (
    <Suspense fallback={<ProfileSkeleton />}>
      <UserProfile />
    </Suspense>
  );
}

// Server component - data loaded before rendering
async function UserProfile() {
  const user = await getUser();
  return <Profile user={user} />;
}
```

**Astro - Already server-first by default:**

```astro
---
// Data fetched at build time - zero client JS, zero loading states
const page = await getPage('homepage');
---
<h1>{page.title}</h1>
```

### Decision Checklist

Before adding a loading state, ask:

1. **Is the loading state in the right place?** It should show BEFORE the page renders, not after
2. **Am I using a loader/server component?** If not, the skeleton will flash
3. **Is the loading state covering data that was already rendered?** If yes, you have a hydration problem
4. **Does this component need client-side interactivity?** If no, keep it server-rendered

### When Loading States ARE Appropriate

- **During navigation** (pendingComponent/Suspense while loader runs)
- **User-triggered actions** (submitting a form, clicking a button)
- **Genuinely slow external APIs** (third-party services that take 3+ seconds)
- **Infinite scroll / pagination** (loading more content)
- **Real-time data updates** (polling, WebSocket)
- **Below-the-fold content** (lazy load with Intersection Observer)

### When Client-Side Fetching is NOT Appropriate

- **Initial page data** (dashboard stats, user profile, page content) - use loaders
- **Data that's available in the database** (query it in the loader, not in useEffect)
- **Auth-gated content** (check auth in beforeLoad/middleware, not after render)

### Caching: Skeleton Only on First Visit

The skeleton should only appear on the **first** navigation to a page. Returning to the same page should be instant from cache.

- **TanStack Start:** Set `staleTime` on route loaders to cache data. Use `<Link preload="intent">` to prefetch on hover. See `framework/tanstack-start` skill for details.
- **Astro:** Static HTML means zero loading states. Enable View Transitions + prefetching for SPA-like instant navigation. See `framework/astro` skill.
- **Next.js:** Use `fetch()` with `revalidate` in Server Components, or React Query with `staleTime` on the client.

### Key Principle

**Skeletons are good. Skeletons that replace already-rendered content are the problem.** Always load data in loaders/server components so the skeleton shows during navigation, not after the page has already painted. Cache the data so the skeleton only shows once.

## Code Quality Gates

### Pre-Commit Checks

Before allowing commit, verify:

- [ ] No console.log statements (unless approved)
- [ ] No TODO comments without issue reference
- [ ] TypeScript strict mode compliance
- [ ] Related task checkboxes updated
- [ ] Run `npx ultracite fix` for linting

### Quality Standards

- **TypeScript** - Use proper types, avoid `any`
- **Error handling** - Handle error cases appropriately
- **Naming** - Follow existing conventions in codebase
- **Simplicity** - Don't over-engineer solutions

## Pattern Detection

### When to Document Patterns

Suggest creating documentation when:
- Same code pattern used 3+ times
- New technology added to project
- Performance optimization discovered
- Security fix applied

**Example suggestion:**
```
I notice you're using this API pattern repeatedly.
Should I document it in the relevant skill file for consistency?
```

## Progress Tracking

### Velocity Awareness

Track informally:
- Tasks completed per session
- Blockers encountered
- Estimation accuracy

### Lessons Learned

After completing complex tasks, note:
- What worked well
- What took longer than expected
- Patterns to reuse

## Quick Commands

Users can say:
- "status" → Show current task status
- "next" → Suggest next task
- "commit" → Create commit with proper message
- "wrap up" → End session protocol
- "blockers" → List blocked tasks

## Key Principles

1. **Proactive updates** - Update tasks as you work, not just at the end
2. **Clear commits** - Each commit should be self-documenting
3. **Context preservation** - Always update dev-context for in-progress work
4. **Pattern consistency** - Check existing code before creating new patterns
5. **Simplicity** - Don't over-engineer, keep solutions focused
