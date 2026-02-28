# RLS Performance

RLS policies execute for every row. Poorly written policies can cause exponential query times. Follow these patterns for performant RLS.

## Always Wrap Functions in SELECT

**Bad - function called per row:**
```sql
CREATE POLICY "Users can view own posts" ON posts
FOR SELECT USING (auth.uid() = user_id);
```

**Good - function result cached:**
```sql
CREATE POLICY "Users can view own posts" ON posts
FOR SELECT USING ((SELECT auth.uid()) = user_id);
```

The `(SELECT ...)` wrapper lets Postgres cache the function result instead of recalculating for every row.

## Index Columns Used in Policies

Always create indexes on columns referenced in RLS policies:

```sql
-- If your policy checks user_id
CREATE INDEX idx_posts_user_id ON posts(user_id);

-- If your policy checks organization_id
CREATE INDEX idx_posts_org_id ON posts(organization_id);
```

## Never Pass Row Data to Functions

**Bad - function called per row with row data:**
```sql
CREATE POLICY "check_access" ON documents
FOR SELECT USING (check_user_access(id, user_id));  -- SLOW!
```

**Good - use direct comparisons:**
```sql
CREATE POLICY "check_access" ON documents
FOR SELECT USING ((SELECT auth.uid()) = user_id);
```

Functions receiving row data cannot be cached and execute once per row.

## Avoid Subqueries When Possible

Subqueries in policies execute for every row:

**Slow - subquery per row:**
```sql
CREATE POLICY "org_members" ON documents
FOR SELECT USING (
  EXISTS (
    SELECT 1 FROM org_members
    WHERE org_id = documents.org_id
    AND user_id = (SELECT auth.uid())
  )
);
```

**Faster - use SECURITY DEFINER function:**
```sql
-- Create a function that bypasses RLS
CREATE OR REPLACE FUNCTION get_user_org_ids()
RETURNS SETOF uuid AS $$
  SELECT org_id FROM org_members WHERE user_id = auth.uid()
$$ LANGUAGE sql SECURITY DEFINER STABLE;

-- Use it in policy
CREATE POLICY "org_members" ON documents
FOR SELECT USING (org_id IN (SELECT get_user_org_ids()));
```

## Consider Denormalization

For high-traffic tables, denormalize the user_id directly onto the row instead of joining:

```sql
-- Instead of joining through organizations
-- Add user_id directly to the documents table
ALTER TABLE documents ADD COLUMN owner_user_id uuid;

-- Simple, fast policy
CREATE POLICY "owner_access" ON documents
FOR SELECT USING ((SELECT auth.uid()) = owner_user_id);
```

## Test RLS Performance

Enable query analysis to debug slow policies:

```sql
-- Enable EXPLAIN for PostgREST
ALTER ROLE authenticator SET pgrst.db_plan_enabled TO true;
NOTIFY pgrst, 'reload config';
```

Then use `.explain()` in your Supabase client to see query plans.

## ACL Storage Strategy

For complex permissions (sharing with users/groups), where you store the ACL matters:

**Option 1: ACL in column (fastest)**
```sql
-- Store permitted user IDs directly on the row
ALTER TABLE documents ADD COLUMN read_access uuid[] DEFAULT '{}';
ALTER TABLE documents ADD COLUMN write_access uuid[] DEFAULT '{}';

-- Add GIN index for array operations
CREATE INDEX idx_docs_read ON documents USING GIN(read_access);

-- Fast policy using array overlap
CREATE POLICY "shared_access" ON documents
FOR SELECT USING (
  read_access && ARRAY[(SELECT auth.uid())]
);
```

**Option 2: ACL in separate table (slower but normalized)**
```sql
-- Separate permissions table
CREATE TABLE document_permissions (
  document_id uuid REFERENCES documents(id),
  user_id uuid,
  role text  -- 'read', 'write', 'owner'
);

-- Policy with subquery (slower)
CREATE POLICY "shared_access" ON documents
FOR SELECT USING (
  EXISTS (
    SELECT 1 FROM document_permissions
    WHERE document_id = documents.id
    AND user_id = (SELECT auth.uid())
  )
);
```

| Approach | Speed | Trade-off |
|----------|-------|-----------|
| ACL in column | ~1ms | Denormalized, wide rows if many permissions |
| ACL in table | ~20ms+ | Normalized, but slower JOINs |

**Recommendation:** For most apps, use column-based ACL. Only use a separate table if you need complex permission hierarchies or audit trails.

## Avoid OR with Boolean Columns

Adding `OR public = TRUE` to policies forces sequential scans:

```sql
-- Bad - forces sequential scan
CREATE POLICY "public_or_owner" ON items
FOR SELECT USING (
  public = TRUE OR (SELECT auth.uid()) = user_id
);

-- Better - use a "public" group ID instead
-- Add the public group ID to the ACL array
```

## Performance Checklist

| Pattern | Performance |
|---------|-------------|
| `(SELECT auth.uid()) = user_id` | Fast - cached |
| `auth.uid() = user_id` | Slow - per row |
| Direct column comparison | Fast |
| Subquery per row | Slow |
| SECURITY DEFINER function | Fast |
| Function with row data param | Slow |
| Indexed columns in policy | Fast |
| Non-indexed columns | Slow |
| ACL in column with GIN index | Fast |
| ACL in separate table | Slow |
| `OR public = TRUE` | Slow - sequential scan |
