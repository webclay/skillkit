# Realtime Subscriptions

Supabase includes built-in real-time sync. Use this when multiple users need to see updates instantly (collaborative features, live dashboards, chat).

## Subscribe to Table Changes

```ts
import { createClient } from '@/lib/supabase/client'

const supabase = createClient()

// Subscribe to all changes on a table
const channel = supabase
  .channel('contacts-changes')
  .on(
    'postgres_changes',
    { event: '*', schema: 'public', table: 'contacts' },
    (payload) => {
      console.log('Change:', payload.eventType, payload.new)
      // Update your local state/UI here
    }
  )
  .subscribe()

// Cleanup when component unmounts
return () => {
  supabase.removeChannel(channel)
}
```

## Filter by Specific Events

```ts
// Only inserts
.on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'messages' }, handler)

// Only updates
.on('postgres_changes', { event: 'UPDATE', schema: 'public', table: 'contacts' }, handler)

// Only deletes
.on('postgres_changes', { event: 'DELETE', schema: 'public', table: 'contacts' }, handler)
```

## Filter by Row

```ts
// Only changes to a specific organization's contacts
.on(
  'postgres_changes',
  {
    event: '*',
    schema: 'public',
    table: 'contacts',
    filter: `organization_id=eq.${orgId}`
  },
  handler
)
```

## React Hook Pattern

```tsx
import { useEffect, useState } from 'react'
import { createClient } from '@/lib/supabase/client'

function useRealtimeContacts(orgId: string) {
  const [contacts, setContacts] = useState<Contact[]>([])
  const supabase = createClient()

  useEffect(() => {
    // Initial fetch
    supabase
      .from('contacts')
      .select('*')
      .eq('organization_id', orgId)
      .then(({ data }) => setContacts(data || []))

    // Subscribe to changes
    const channel = supabase
      .channel(`contacts-${orgId}`)
      .on(
        'postgres_changes',
        {
          event: '*',
          schema: 'public',
          table: 'contacts',
          filter: `organization_id=eq.${orgId}`
        },
        (payload) => {
          if (payload.eventType === 'INSERT') {
            setContacts(prev => [...prev, payload.new as Contact])
          } else if (payload.eventType === 'UPDATE') {
            setContacts(prev =>
              prev.map(c => c.id === payload.new.id ? payload.new as Contact : c)
            )
          } else if (payload.eventType === 'DELETE') {
            setContacts(prev => prev.filter(c => c.id !== payload.old.id))
          }
        }
      )
      .subscribe()

    return () => {
      supabase.removeChannel(channel)
    }
  }, [orgId])

  return contacts
}
```

## Enable Realtime on a Table (Required)

```sql
-- In Supabase SQL Editor or migration
ALTER PUBLICATION supabase_realtime ADD TABLE contacts;
```

Or enable via Dashboard: Table Editor -> Select table -> Enable Realtime

## When to Use Realtime

| Use case | Realtime needed? |
|----------|------------------|
| Multiple users editing same data | Yes |
| Live dashboards/analytics | Yes |
| Chat/messaging features | Yes |
| Single user CRUD | No - standard queries are fine |
| Data that rarely changes | No |

## Realtime vs Electric SQL

If you need real-time sync, Supabase Realtime is the simplest choice when using Supabase. You don't need Electric SQL - that's for self-hosted Postgres without Supabase.
