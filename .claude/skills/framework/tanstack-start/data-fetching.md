# Data Fetching Strategy

**Start with loaders. They're like a mini-Query.** If you need more control, move to TanStack Query.

| Approach | When to Use |
|----------|-------------|
| **Loaders** (default) | Route-level data, initial page load, simple fetch-and-display |
| **TanStack Query** | Mutations, optimistic updates, infinite scroll, manual refetching, shared cache across components, polling |
| **API Routes** | Database operations (Prisma), authentication, webhooks |

**Progression path:**
1. Start with loaders for all route data
2. Add TanStack Query when you need mutations or cross-component cache
3. Use API routes for server-only code (DB, auth)

```tsx
// Step 1: Simple loader (start here)
export const Route = createFileRoute('/posts')({
  loader: async () => {
    const posts = await fetch('/api/posts').then(r => r.json());
    return { posts };
  },
  component: Posts,
});

// Step 2: Graduate to Query when you need mutations
import { useMutation, useQueryClient } from '@tanstack/react-query';

function Posts() {
  const { posts } = Route.useLoaderData();
  const queryClient = useQueryClient();

  const deleteMutation = useMutation({
    mutationFn: (id: string) => fetch(`/api/posts/${id}`, { method: 'DELETE' }),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['posts'] }),
  });

  return /* ... */;
}
```

## Preventing UI Flash / Double-Load (Critical)

**Never use `useEffect` + `useState` for initial page data.** This causes the classic flash: content renders on server, disappears into a skeleton, then reappears after client-side fetch.

```tsx
// BAD - causes flash/double-load
function Dashboard() {
  const [stats, setStats] = useState(null);
  const [projects, setProjects] = useState([]);

  useEffect(() => {
    fetch('/api/dashboard').then(r => r.json()).then(setStats);
    fetch('/api/projects').then(r => r.json()).then(d => setProjects(d.projects));
  }, []);

  if (!stats) return <DashboardSkeleton />;  // User sees flash here
  return <DashboardContent stats={stats} projects={projects} />;
}

// GOOD - skeleton shows during navigation, page renders once with data
export const Route = createFileRoute('/dashboard')({
  loader: async () => {
    const [stats, projects] = await Promise.all([
      fetch('/api/dashboard').then(r => r.json()),
      fetch('/api/projects').then(r => r.json()),
    ]);
    return { stats, projects: projects.projects };
  },
  pendingComponent: () => <DashboardSkeleton />,  // Shows DURING navigation (correct)
  component: Dashboard,
});

function Dashboard() {
  const { stats, projects } = Route.useLoaderData();
  // Data is ready - page renders once, no second loading state
  return <DashboardContent stats={stats} projects={projects} />;
}
```

**Rules:**

1. **All initial page data goes in `loader`** - The page should render complete on first paint
2. **Use `pendingComponent` for navigation loading** - Skeleton shows BEFORE the page renders (during loader execution), not after. This is the correct skeleton pattern
3. **Parallel fetch in loaders** - Use `Promise.all()` to avoid waterfalls
4. **Reserve client-side fetching for user actions** - Form submissions, infinite scroll, real-time updates
5. **Auth checks in `beforeLoad`** - Don't render a page then redirect after checking auth on the client

```tsx
// GOOD - auth checked before render, no flash of unauthorized content
export const Route = createFileRoute('/dashboard')({
  beforeLoad: async () => {
    const session = await getSession();
    if (!session) throw redirect({ to: '/login' });
    return { session };
  },
  loader: async ({ context }) => {
    const data = await fetchDashboardData(context.session.userId);
    return { data };
  },
  component: Dashboard,
});
```

## Loader Caching (Skeleton Only on First Visit)

**The skeleton should only show on the first navigation.** Subsequent visits to the same page should be instant from cache. TanStack Router supports this via `staleTime`.

```tsx
// Skeleton shows on first visit, cached for 5 minutes after
export const Route = createFileRoute('/dashboard')({
  loader: async () => {
    const [stats, projects] = await Promise.all([
      fetch('/api/dashboard').then(r => r.json()),
      fetch('/api/projects').then(r => r.json()),
    ]);
    return { stats, projects: projects.projects };
  },
  pendingComponent: () => <DashboardSkeleton />,
  staleTime: 5 * 60 * 1000,  // Data stays fresh for 5 minutes
  component: Dashboard,
});
```

**How it works:**
- First visit to `/dashboard`: loader runs, skeleton shows, page renders with data
- Navigate to `/settings` then back to `/dashboard` (within 5 min): instant, no skeleton, cached data served
- After 5 minutes: data is stale, next visit re-runs loader with skeleton

**Common staleTime values:**
- `0` (default) - Always refetch on navigation (skeleton every time)
- `30_000` (30s) - Good for frequently changing data
- `5 * 60 * 1000` (5 min) - Good for dashboards, profiles
- `Infinity` - Never refetch (only load once per session)

**For more control, use TanStack Query in the loader:**

```tsx
// TanStack Query gives you staleTime + background refetching + shared cache
export const Route = createFileRoute('/dashboard')({
  loader: async ({ context }) => {
    // ensureQueryData uses cache if fresh, fetches if stale
    const stats = await context.queryClient.ensureQueryData({
      queryKey: ['dashboard-stats'],
      queryFn: fetchDashboardStats,
      staleTime: 5 * 60 * 1000,
    });
    return { stats };
  },
  pendingComponent: () => <DashboardSkeleton />,
  component: Dashboard,
});

function Dashboard() {
  // useLoaderData for initial data, useSuspenseQuery for reactive updates
  const { stats } = Route.useLoaderData();
  return <DashboardContent stats={stats} />;
}
```

**When to use which:**

| Approach | Use when |
|----------|----------|
| `staleTime` on route | Simple caching, data only needed on this route |
| TanStack Query `ensureQueryData` | Same data used across multiple routes, need background refetching, need manual invalidation |

**Preloading on hover (instant navigation):**

```tsx
// TanStack Router can preload routes on hover/intent
<Link to="/dashboard" preload="intent">Dashboard</Link>
```

When the user hovers over the link, the loader starts running. By the time they click, data is already cached - zero skeleton, zero wait.

**See also:** `workflow/best-practices` skill for the full Server-First Rendering guide.
