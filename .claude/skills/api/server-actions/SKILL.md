---
name: server-actions
description: Next.js Server Actions and API route patterns. Trigger words - server action, use server, form action, api route, mutation, server function
---

# Next.js Server Actions

Secure server-side logic patterns for Next.js 15 with authentication, validation, and error handling.

## When to Use This Skill

- User asks about Server Actions or API routes
- User needs secure form submissions
- User mentions data mutations or CRUD operations
- User asks about backend security patterns

## Server Action Pattern

**app/actions/user.ts:**
```typescript
'use server';

import { z } from 'zod';
import { revalidatePath } from 'next/cache';
import { db } from '@/lib/db';
import { auth } from '@/lib/auth';

const createUserSchema = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
});

export async function createUser(formData: FormData) {
  // 1. Authenticate
  const session = await auth();
  if (!session?.user) {
    return { success: false, error: 'Unauthorized' };
  }

  // 2. Validate input
  const parsed = createUserSchema.safeParse({
    name: formData.get('name'),
    email: formData.get('email'),
  });

  if (!parsed.success) {
    return { success: false, error: 'Invalid input' };
  }

  // 3. Execute with try/catch
  try {
    const user = await db.user.create({
      data: parsed.data
    });

    revalidatePath('/users');
    return { success: true, data: user };
  } catch (error) {
    console.error('Create user error:', error);
    return { success: false, error: 'Failed to create user' };
  }
}
```

## API Route Pattern

**app/api/items/route.ts:**
```typescript
import { NextResponse } from 'next/server';
import { auth } from '@/lib/auth';
import { db } from '@/lib/db';

export async function GET() {
  // 1. Authenticate
  const session = await auth();
  if (!session?.user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  // 2. Fetch data
  try {
    const items = await db.item.findMany({
      where: { userId: session.user.id }
    });

    return NextResponse.json({ success: true, items });
  } catch (error) {
    return NextResponse.json({ error: 'Failed to fetch' }, { status: 500 });
  }
}

export async function POST(request: Request) {
  const session = await auth();
  if (!session?.user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const body = await request.json();

  try {
    const item = await db.item.create({
      data: { ...body, userId: session.user.id }
    });

    return NextResponse.json({ success: true, item });
  } catch (error) {
    return NextResponse.json({ error: 'Failed to create' }, { status: 500 });
  }
}

export async function DELETE(request: Request) {
  const session = await auth();
  if (!session?.user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const { searchParams } = new URL(request.url);
  const id = searchParams.get('id');

  if (!id) {
    return NextResponse.json({ error: 'ID required' }, { status: 400 });
  }

  try {
    await db.item.delete({
      where: { id, userId: session.user.id }  // Always filter by user
    });

    return NextResponse.json({ success: true });
  } catch (error) {
    return NextResponse.json({ error: 'Failed to delete' }, { status: 500 });
  }
}
```

## Using Server Actions in Forms

```tsx
import { createUser } from '@/app/actions/user';

export function CreateUserForm() {
  return (
    <form action={createUser}>
      <input name="name" required />
      <input name="email" type="email" required />
      <button type="submit">Create</button>
    </form>
  );
}
```

## Security Rules

1. **Always validate inputs** with Zod at the start
2. **Always authenticate** before any data operation
3. **Always filter by userId** in queries (even with RLS)
4. **Never expose secrets** to client-side code
5. **Return consistent shapes**: `{ success: true, data }` or `{ success: false, error }`

## Response Status Codes

- `200`: Success
- `201`: Created
- `400`: Bad request (validation)
- `401`: Unauthorized (not logged in)
- `403`: Forbidden (logged in but not allowed)
- `404`: Not found
- `500`: Server error

## Tips

- Use `revalidatePath()` or `revalidateTag()` after mutations
- Keep server actions in separate files with `'use server'`
- Log errors server-side, return sanitized messages to client
- Use `useActionState` for form state management

## How to Verify

### Quick Checks
- Form submission triggers server action
- Protected routes return 401 when unauthenticated
- Invalid data returns validation errors

### Common Issues
- "Unauthorized": Check auth session is valid
- Action not firing: Ensure `'use server'` directive is present
- Data not updating: Call `revalidatePath()` after mutation
