---
name: nextjs
description: Next.js App Router patterns and best practices. Trigger words - nextjs, next.js, app router, page, layout, server component, client component, server action, api route, middleware, loading, error boundary, use server, use client
---

# Next.js

React framework with App Router, Server Components, Server Actions, and built-in optimizations.

## When to Use This Skill

- User asks about Next.js patterns
- Project has `next.config.js` or `app/` directory
- User mentions App Router, server components, or server actions
- User asks about API routes or middleware
- Working with React Server Components

## Instructions

### Project Structure

```
app/
├── layout.tsx           # Root layout
├── page.tsx             # Home page (/)
├── loading.tsx          # Loading UI
├── error.tsx            # Error boundary
├── not-found.tsx        # 404 page
├── api/                 # API routes
│   └── route.ts
├── dashboard/
│   ├── page.tsx         # /dashboard
│   └── [id]/
│       └── page.tsx     # /dashboard/:id
└── (auth)/              # Route group (no URL segment)
    ├── login/page.tsx
    └── signup/page.tsx
```

### Server Components (Default)

```tsx
// app/posts/page.tsx - Server Component
async function PostsPage() {
  const posts = await db.post.findMany();  // Direct DB access

  return (
    <ul>
      {posts.map(post => <li key={post.id}>{post.title}</li>)}
    </ul>
  );
}

export default PostsPage;
```

### Client Components

```tsx
'use client';

import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

### Server Actions

```tsx
// app/actions.ts
'use server';

import { revalidatePath } from 'next/cache';

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  await db.post.create({ data: { title } });
  revalidatePath('/posts');
}
```

```tsx
// app/posts/new/page.tsx
import { createPost } from '../actions';

export default function NewPost() {
  return (
    <form action={createPost}>
      <input name="title" required />
      <button type="submit">Create</button>
    </form>
  );
}
```

## Examples

**API Route:**
```ts
// app/api/posts/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
  const posts = await db.post.findMany();
  return NextResponse.json(posts);
}

export async function POST(request: Request) {
  const data = await request.json();
  const post = await db.post.create({ data });
  return NextResponse.json(post);
}
```

**Dynamic Route:**
```tsx
// app/posts/[id]/page.tsx
export default async function PostPage({ params }: { params: { id: string } }) {
  const post = await db.post.findUnique({ where: { id: params.id } });
  if (!post) notFound();
  return <h1>{post.title}</h1>;
}
```

**Layout:**
```tsx
// app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

**Metadata:**
```tsx
export const metadata = {
  title: 'My App',
  description: 'App description',
};

// Dynamic metadata
export async function generateMetadata({ params }: { params: { id: string } }) {
  const post = await getPost(params.id);
  return { title: post.title };
}
```

**Middleware:**
```ts
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const isLoggedIn = request.cookies.get('session');
  if (!isLoggedIn && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
}

export const config = { matcher: ['/dashboard/:path*'] };
```

**Loading & Error:**
```tsx
// app/posts/loading.tsx
export default function Loading() {
  return <div>Loading posts...</div>;
}

// app/posts/error.tsx
'use client';
export default function Error({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div>
      <p>Error: {error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

## Reference Files

- [routing.md](routing.md) - Advanced routing patterns
- [caching.md](caching.md) - Caching strategies

## Key Patterns

| Pattern | Use |
|---------|-----|
| Server Component | Data fetching, DB access |
| Client Component | Interactivity, hooks, browser APIs |
| Server Action | Form submissions, mutations |
| Route Handler | REST APIs, webhooks |

## Tips

- Default to Server Components, add `'use client'` only when needed
- Use `revalidatePath()` or `revalidateTag()` after mutations
- Colocate loading.tsx and error.tsx with page.tsx
- Use route groups `(folder)` to organize without affecting URL

## How to Verify

### Quick Checks
- `npm run build` - Build completes without errors
- `npm run dev` - Dev server starts and page loads
- No TypeScript errors in terminal

### Manual Verification
- Navigate to new route - page renders correctly
- API routes return expected data (`curl localhost:3000/api/...`)
- Server actions trigger and revalidate correctly
- Loading states appear during data fetching

### Common Issues
- "Module not found": Check import paths use `@/` alias correctly
- "Text content mismatch": Hydration error - ensure server/client render same content
- "use client" errors: Only client components can use hooks/state
- API route 404: Check file is named `route.ts` not `route.tsx`
