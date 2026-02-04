---
name: electric-sql
description: Electric SQL for real-time Postgres sync to local apps. Trigger words - electric, electricsql, local-first, offline support, offline-first, sync, continuous sync, local database
---

# Electric SQL

Postgres sync engine for local-first applications with real-time sync and offline support.

## When to Use This Skill

- Building local-first apps with offline support
- Need real-time data sync from self-hosted Postgres
- Want to eliminate loading states with local data
- Building collaborative or multiplayer apps
- Replacing REST APIs with continuous sync

## When NOT to Use

- Simple CRUD without offline needs
- Server-only data processing
- Non-Postgres databases
- **Using Supabase** - use Supabase Realtime instead (see below)

## Important: Supabase Projects

**If your project uses Supabase, do NOT use Electric SQL.** Use Supabase Realtime instead - it's built-in, simpler, and requires no extra infrastructure.

See the [Supabase skill](../supabase/SKILL.md) for Realtime setup.

| Feature | Electric SQL | Supabase Realtime |
|---------|--------------|-------------------|
| Real-time sync | Yes | Yes |
| Offline support | Yes | No |
| Self-hosted | Required | Managed by Supabase |
| Setup complexity | Docker + config | Just enable per table |

**Choose Electric SQL only if:**
- You need offline/local-first capability
- You're using self-hosted Postgres (not Supabase)
- You want data replicated locally on the client

## Setup

```bash
npm i @electric-sql/client @electric-sql/react
```

### Environment Variables

```env
ELECTRIC_URL=http://localhost:3000  # Electric sync service
DATABASE_URL=postgresql://...       # Your Postgres database
```

## Core Concept: Shapes

Shapes define subsets of Postgres data to sync:

```typescript
// Sync all items
{ table: 'items' }

// Sync filtered items
{ table: 'items', where: `user_id = '${userId}'` }

// Sync specific columns
{ table: 'items', columns: ['id', 'title', 'status'] }
```

## React Integration

```tsx
import { useShape } from '@electric-sql/react'

function ItemList() {
  const { data, isLoading, isError } = useShape({
    url: 'http://localhost:3000/v1/shape',
    params: {
      table: 'items',
      where: `status = 'active'`
    }
  })

  if (isLoading) return <div>Loading...</div>
  if (isError) return <div>Error loading data</div>

  return (
    <ul>
      {data.map((item) => (
        <li key={item.id}>{item.title}</li>
      ))}
    </ul>
  )
}
```

## API Proxy Pattern (Required for Production)

Always proxy Electric requests through your backend for security:

```typescript
// app/api/items/route.ts (Next.js)
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url)

  // Authenticate user
  const session = await getSession()
  if (!session) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  // Build Electric URL with user filtering
  const electricUrl = new URL(`${process.env.ELECTRIC_URL}/v1/shape`)
  electricUrl.searchParams.set('table', 'items')
  electricUrl.searchParams.set('where', `user_id = '${session.user.id}'`)

  // Forward protocol params
  ;['offset', 'handle', 'live', 'cursor'].forEach((param) => {
    const value = searchParams.get(param)
    if (value) electricUrl.searchParams.set(param, value)
  })

  // Proxy request
  const response = await fetch(electricUrl.toString())
  return new NextResponse(response.body, {
    status: response.status,
    headers: response.headers
  })
}
```

Then use from React:

```tsx
const { data } = useShape({
  url: '/api/items'  // Your proxy endpoint
})
```

## Low-Level ShapeStream

For custom sync logic:

```typescript
import { ShapeStream, Shape } from '@electric-sql/client'

const stream = new ShapeStream({
  url: `http://localhost:3000/v1/shape`,
  params: { table: 'items' }
})

const shape = new Shape(stream)

// Wait for initial data
await shape.rows

// Subscribe to changes
shape.subscribe(({ rows }) => {
  console.log('Current data:', rows)
})
```

## Optimistic Updates Pattern

```typescript
function useOptimisticItems() {
  const { data: serverItems } = useShape({ url: '/api/items' })
  const [optimistic, setOptimistic] = useState(new Map())

  const items = useMemo(() => {
    const merged = [...serverItems]
    optimistic.forEach((update, id) => {
      const idx = merged.findIndex((i) => i.id === id)
      if (idx >= 0) merged[idx] = update
      else merged.push(update)
    })
    return merged
  }, [serverItems, optimistic])

  const updateItem = async (id, changes) => {
    // Apply optimistically
    const current = items.find((i) => i.id === id)
    setOptimistic((prev) => new Map(prev).set(id, { ...current, ...changes }))

    try {
      await fetch(`/api/items/${id}`, {
        method: 'PATCH',
        body: JSON.stringify(changes)
      })
      // Server update syncs back automatically
    } catch {
      // Rollback on error
      setOptimistic((prev) => {
        const next = new Map(prev)
        next.delete(id)
        return next
      })
    }
  }

  return { items, updateItem }
}
```

## Docker Deployment

```yaml
# docker-compose.yml
services:
  electric:
    image: electricsql/electric
    environment:
      DATABASE_URL: postgresql://user:pass@postgres:5432/db
      ELECTRIC_SECRET: your-secret-key
    ports:
      - "3000:3000"

  postgres:
    image: postgres:16
    command: postgres -c wal_level=logical
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: db
```

## Tips

- Use API proxy pattern in production (never expose Electric directly)
- Shapes are immutable after subscription starts
- Postgres v14+ required with logical replication enabled
- Electric optimizes `field = constant` where clauses

## How to Verify

### Quick Checks
- Electric health: `curl http://localhost:3000/v1/health`
- useShape returns data array (not undefined after load)
- Changes in Postgres appear in client automatically

### Common Issues
- No sync: Check Postgres has `wal_level=logical`
- Unauthorized: Verify API proxy handles auth correctly
- Shape not updating: Shapes are immutable, create new subscription
