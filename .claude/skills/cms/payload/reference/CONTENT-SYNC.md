# Content Sync (Local to Live)

Reference for pushing local content to a live Payload backend. Solves the gap where `onInit` seeding only works on empty databases.

## The Problem

Content website templates use an `onInit` hook to seed initial content (homepage, globals) on first startup. This hook checks if content exists and skips seeding if it does:

```typescript
// Typical onInit seed guard
const existing = await payload.find({ collection: 'pages', where: { slug: { equals: 'home' } } })
if (!existing.docs.length || existing.docs[0].hero?.title === 'Hero Header') {
  // Seed content...
}
```

**This means:** Once the live database has content (even placeholder content from the first deploy), updating `onInit` seed data and redeploying does nothing. The guard sees existing content and skips.

**The user's workflow:** Develop content locally (update seed data, test with local Payload) then want that content on the live backend - without manually editing every field in the admin panel.

## When to Use

- Local content development is done and needs to go live
- Homepage or global content was updated in seed files
- Content needs to be pushed to a deployed backend without admin panel editing
- Any time the user says "push this content to live" or "update the live site with this content"

## When NOT to Use

- First deploy (onInit seeding handles this)
- Content changes that should go through the admin panel (editorial workflow)
- Schema/collection changes (use migrations instead)

## Solution: Local API Script (Recommended)

A one-time script that boots Payload against the target database and uses the Local API to update content. No running backend needed, no authentication needed - just the `DATABASE_URI`.

### Complete Script

```typescript
// scripts/update-homepage-content.ts
//
// Updates homepage content on a live database using Payload's local API.
//
// Usage:
//   cd backend
//   DATABASE_URI="postgresql://..." pnpm dlx tsx scripts/update-homepage-content.ts

async function main() {
  if (!process.env.DATABASE_URI) {
    console.error('DATABASE_URI is required.')
    console.error('')
    console.error('Usage:')
    console.error('  DATABASE_URI="postgresql://..." pnpm dlx tsx scripts/update-homepage-content.ts')
    process.exit(1)
  }

  // Set required env vars so payload.config.ts doesn't fail
  process.env.PAYLOAD_SECRET = process.env.PAYLOAD_SECRET || 'local-script-secret'
  process.env.PAYLOAD_PUBLIC_SERVER_URL = process.env.PAYLOAD_PUBLIC_SERVER_URL || 'http://localhost:3000'

  console.log('Booting Payload...')
  const { getPayload } = await import('payload')
  const { homePageSeedDE } = await import('../src/seed/homePageSeed')

  const payload = await getPayload({
    config: (await import('../src/payload.config')).default,
  })

  // Update the Pages collection document (slug: 'home')
  console.log('Finding homepage in Pages collection...')
  const pagesResult = await payload.find({
    collection: 'pages',
    where: { slug: { equals: 'home' } },
    locale: 'de',
    limit: 1,
  })

  if (pagesResult.docs.length > 0) {
    const pageId = pagesResult.docs[0].id
    console.log(`Updating Page (id: ${pageId})...`)
    await payload.update({
      collection: 'pages',
      id: pageId,
      locale: 'de',
      draft: false,
      data: { title: 'Startseite', ...homePageSeedDE } as any,
    })
    console.log('Pages document updated.')
  }

  // Update the home-page global
  console.log('Updating home-page global...')
  await payload.updateGlobal({
    slug: 'home-page',
    locale: 'de',
    draft: false,
    data: homePageSeedDE as any,
  })
  console.log('Home-page global updated.')

  console.log('\nDone! Content updated.')
  process.exit(0)
}

main().catch((err) => {
  console.error('Script failed:', err)
  process.exit(1)
})
```

### Running It

```bash
cd backend
DATABASE_URI="postgresql://user:pass@host:port/db" pnpm dlx tsx scripts/update-homepage-content.ts
```

The `DATABASE_URI` is the same connection string used by Railway. Find it in Railway > your backend service > Variables > `DATABASE_URI`.

**Why Local API over REST API:**
- No running backend needed - boots Payload locally against the remote database
- No authentication needed - Local API bypasses access control
- Simpler script - no login flow, no auth tokens, no HTTP requests
- Handles version tables automatically - Payload manages `_pages_v` and `_pages_v_locales` internally

**After running:** Delete the script - it's a one-time tool. If content changes again, regenerate it from the updated seed data.

## How to Generate the Script

When the user asks to push content to live, follow this process:

1. **Find the seed data** - Look for `onInit` hooks in `payload.config.ts`, or seed files in `backend/src/seed/`, `backend/seed/`, or similar locations
2. **Read the content** - Understand the data structure (collections, globals, fields, locales)
3. **Generate the sync script** - Create a TypeScript file that:
   - Requires `DATABASE_URI` as an environment variable
   - Sets dummy values for other required env vars (`PAYLOAD_SECRET`, `PAYLOAD_PUBLIC_SERVER_URL`)
   - Boots Payload with the project's config
   - Imports the seed data directly
   - Finds existing documents by slug and updates them via Local API
   - Logs progress and errors clearly
4. **Tell the user how to run it:**
   ```bash
   cd backend
   DATABASE_URI="postgresql://..." pnpm dlx tsx scripts/update-homepage-content.ts
   ```
5. **Remind them to clean up** - The script is a one-time tool, safe to delete after use

### Package manager for running scripts

Use `pnpm dlx tsx` instead of `npx tsx` - it handles temporary packages more reliably. If the project uses bun, `bunx tsx` also works.

```bash
# Preferred
pnpm dlx tsx scripts/update-homepage-content.ts

# Also works
bunx tsx scripts/update-homepage-content.ts
npx tsx scripts/update-homepage-content.ts
```

## Alternative: REST API Script

If the backend is already running and you prefer not to connect directly to the database, you can use the REST API instead. This requires admin credentials and the backend URL.

### Authentication

```typescript
const loginRes = await fetch(`${backendUrl}/api/users/login`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    email: process.env.PAYLOAD_ADMIN_EMAIL || 'admin@example.com',
    password: process.env.PAYLOAD_ADMIN_PASSWORD || 'changeme123',
  }),
})
const { token } = await loginRes.json()
```

### Updating a Collection Document

```typescript
// Find page by slug
const findRes = await fetch(
  `${backendUrl}/api/pages?where[slug][equals]=home&depth=0`,
  { headers: { Authorization: `Bearer ${token}` } }
)
const { docs } = await findRes.json()

if (docs.length > 0) {
  await fetch(`${backendUrl}/api/pages/${docs[0].id}`, {
    method: 'PATCH',
    headers: {
      'Content-Type': 'application/json',
      Authorization: `Bearer ${token}`,
    },
    body: JSON.stringify(contentData),
  })
}
```

### Updating a Global

```typescript
await fetch(`${backendUrl}/api/globals/home-page`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    Authorization: `Bearer ${token}`,
  },
  body: JSON.stringify(globalData),
})
```

### Handling Locales via REST API

```typescript
// Update German locale
await fetch(`${backendUrl}/api/pages/${id}?locale=de`, {
  method: 'PATCH',
  headers: {
    'Content-Type': 'application/json',
    Authorization: `Bearer ${token}`,
  },
  body: JSON.stringify(contentDataDE),
})
```

**Important:** For globals with localized array fields, see the `updateGlobal` locale bug documented in [api.md](api.md). The REST API PATCH approach does not have this bug - it correctly updates localized fields inside arrays when `?locale=` is specified.

## Common Migration Issues

These bugs are common in content website projects and often surface when running the sync script for the first time.

### 1. payload_migrations table missing DEFAULT on timestamps

The `from_scratch` migration creates `payload_migrations` with `created_at NOT NULL` but no default. Payload's Drizzle adapter doesn't set this field, causing a NOT NULL violation when recording completed migrations.

**Fix in `from_scratch` migration:**
```sql
-- Before (broken)
updated_at timestamp(3) with time zone NOT NULL,
created_at timestamp(3) with time zone NOT NULL

-- After (fixed)
updated_at timestamp(3) with time zone NOT NULL DEFAULT NOW(),
created_at timestamp(3) with time zone NOT NULL DEFAULT NOW()
```

### 2. Non-idempotent enum rename migrations

A migration that renames an enum value (e.g., `'transparent'` to `'shrinking'`) will fail on a fresh database if `from_scratch` already creates the enum with the new value.

**Fix - wrap with existence check:**
```sql
DO $$ BEGIN
  IF EXISTS (
    SELECT 1 FROM pg_enum
    WHERE enumlabel = 'transparent'
    AND enumtypid = 'enum_header_variant'::regtype
  ) THEN
    ALTER TYPE "enum_header_variant" RENAME VALUE 'transparent' TO 'shrinking';
  END IF;
END $$;
```

### 3. Seed migrations missing version locale records

Raw SQL seeds that insert into `pages` and `pages_locales` but skip `_pages_v_locales` will cause page titles to appear empty in the admin panel. Payload reads from the version tables, not the main tables.

**Fix - also insert version locale records:**
```sql
INSERT INTO _pages_v_locales (version_title, _parent_id, _locale)
SELECT 'Kontakt', v.id, 'de'
FROM _pages_v v WHERE v.parent_id = <page_id> AND v.latest = true
AND NOT EXISTS (
  SELECT 1 FROM _pages_v_locales vl
  WHERE vl._parent_id = v.id AND vl._locale = 'de'
);
```

### 4. Media records lost when switching databases

When switching to a new Railway database, the `media` table is empty even though the actual files still exist on Cloudflare R2. Payload shows "no media found" because it has no records pointing to those files.

**Fix - copy media records from the old database to the new one:**

```bash
# 1. Export media records from old database
PGPASSWORD=<old_password> psql -h <host> -p <old_port> -U postgres -d railway \
  -c "SELECT * FROM media ORDER BY id"

# 2. Insert into new database (adjust values per record)
PGPASSWORD=<new_password> psql -h <host> -p <new_port> -U postgres -d railway -c "
INSERT INTO media (id, alt, prefix, updated_at, created_at, url, thumbnail_u_r_l, filename, mime_type, filesize, width, height, focal_x, focal_y)
VALUES
(1, 'Logo', 'media', NOW(), NOW(), '', '', 'logo.png', 'image/png', 6246, 300, 56, 50, 50),
(2, 'Hero Image', 'media', NOW(), NOW(), '', '', 'hero-01.jpg', 'image/jpeg', 383948, 1920, 1080, 50, 50);

-- Reset the sequence to avoid ID conflicts on future uploads
SELECT setval('media_id_seq', (SELECT MAX(id) FROM media));
"
```

**Key insight:** Payload stores file metadata in PostgreSQL, but the actual files live on R2. Switching databases loses the metadata, not the files. Copying the `media` table rows restores the link without re-uploading.

## Key Takeaways

- **`onInit` only seeds empty content.** To update existing content, use the Payload Local API via a one-time script
- **Local API is simpler than REST API** for sync scripts - no auth, no running backend, handles version tables automatically
- **Payload reads from version tables** (`_pages_v`, `_pages_v_locales`), not the main tables. Raw SQL seeds must populate both, but Local API calls handle this automatically
- **`defaultValue` in Payload field schemas** shows content in the admin even with an empty database. This is not database content - it's schema defaults
- **Always use `pnpm payload migrate`** from the backend directory to run migrations. Set `DATABASE_URI` to target a specific database
- **Migrations must be idempotent** when a `from_scratch` migration exists that already includes later changes
