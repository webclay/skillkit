---
name: database-security-audit
description: Audits database security architecture and access patterns. Trigger words - security audit, database security, rls audit, access control, security review, production ready
---

# Database Security Audit

Reviews database security configuration to ensure data is only accessed through properly authenticated backend routes - not directly from frontend code.

## Core Principle

**RLS is not enough.** AI models default to RLS rules, but that's a bad pattern. The secure architecture is:

1. Enable RLS on all tables with NO permissive rules (closes DB to outside world)
2. Only the service role key can access data
3. All data access goes through API routes or server actions
4. Backend validates user authentication before any database operation
5. Invalid tokens return 403 immediately
6. Frontend NEVER directly talks to the database

**Benefits of this approach:**
- Ultimate control over access
- Can update logic without app releases (critical for mobile)
- Can cut access immediately if issues arise
- Fix bugs without redeploying clients

## When to Use This Skill

- User asks to "audit security" or "review database security"
- Before deploying to production
- After setting up Supabase, Prisma, or Drizzle
- User mentions concerns about data access
- User asks about RLS, Row Level Security, or access control
- After adding new database tables
- User asks "is my database secure?"

## Instructions

### Step 1: Identify Database Type

Check `package.json` for:
- `@supabase/supabase-js` - Supabase
- `@prisma/client` - Prisma
- `drizzle-orm` - Drizzle

### Step 2: Audit Architecture

**Critical Question:** Does the frontend ever directly access the database?

Search for red flags in client code:

```bash
# Supabase anti-patterns - frontend calling database directly
grep -rn "createBrowserClient" --include="*.tsx" --include="*.ts" components/ app/
grep -rn "\.from\(" --include="*.tsx" --include="*.ts" components/
grep -rn "\.select\(" --include="*.tsx" --include="*.ts" components/
grep -rn "\.insert\(" --include="*.tsx" --include="*.ts" components/
```

**Good:** All `.from()` calls are in:
- `app/api/` routes
- Server actions (`'use server'` files)
- Files that only run on server

**Bad:** `.from()` calls in:
- `components/` directory
- Client components (`'use client'`)
- Any file imported by client components

### Step 3: Audit Auth Checks

Every API route and server action MUST check auth first:

```typescript
// Required pattern - auth check FIRST
const session = await auth.api.getSession({ headers: request.headers });
if (!session) {
  return new Response(JSON.stringify({ error: 'Unauthorized' }), { status: 403 });
}
```

Search for missing auth checks:

```bash
# Find all API routes
find app/api -name "route.ts" -o -name "route.tsx"

# Find server actions
grep -rn "'use server'" --include="*.ts" --include="*.tsx"
```

For each file found, verify it has auth validation at the top.

### Step 4: Audit User Filtering

Even with auth, queries MUST filter by userId:

```typescript
// Bad - returns all posts
const posts = await db.post.findMany({ where: { published: true } });

// Good - filters by user
const posts = await db.post.findMany({
  where: { published: true, userId: session.user.id }
});
```

### Step 5: Audit Environment Variables

Check for exposed secrets:

```bash
# Find client-exposed variables
grep -rn "NEXT_PUBLIC_" .env*
grep -rn "VITE_" .env*
```

**Safe to expose:**
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY` (only if RLS blocks everything)

**Never expose:**
- `SUPABASE_SERVICE_ROLE_KEY`
- `DATABASE_URL`
- Any key with write access

### Step 6: Generate Report

Output findings in this format:

```markdown
## Database Security Audit Report

### Architecture Assessment
- [ ] All database access through backend: [PASS/FAIL]
- [ ] No frontend-to-database direct access: [PASS/FAIL]

### Authentication
- [ ] All API routes check auth first: [PASS/FAIL]
- [ ] All server actions check auth first: [PASS/FAIL]
- [ ] Invalid tokens return 403: [PASS/FAIL]

### Authorization
- [ ] All queries filter by userId: [PASS/FAIL]
- [ ] Delete operations verify ownership: [PASS/FAIL]
- [ ] Update operations verify ownership: [PASS/FAIL]

### Environment Security
- [ ] Service role key not exposed: [PASS/FAIL]
- [ ] DATABASE_URL not in client code: [PASS/FAIL]

### Critical Issues (must fix)
1. [Issue with file path and line number]
   - **Risk:** [What could go wrong]
   - **Fix:** [How to remediate]

### Warnings (should fix)
1. [Issue description]

### Recommendations
1. [Improvement suggestion]
```

## Examples

**User:** "Is my database secure?"

Run full audit:
1. Check package.json for database type
2. Search for frontend database access
3. Audit all API routes for auth checks
4. Verify userId filtering on queries
5. Check environment variables
6. Generate report with findings

**User:** "Audit my Supabase setup"

1. Verify anon key only used for auth, not data
2. Check RLS is enabled but not relied upon for security
3. Confirm all mutations go through API routes
4. Verify service role key only in server code
5. Check auth.getUser() called before every database operation

## Reference Files

- [anti-patterns.md](anti-patterns.md) - Code patterns to flag during audit
- [checklists.md](checklists.md) - Database-specific audit checklists
- [remediation.md](remediation.md) - How to fix common security issues

## Critical Rules

1. **Never trust RLS alone** - It's a safety net, not the security layer
2. **Always check auth first** - Every API route, every server action
3. **Filter by userId** - Even with auth, always add `where: { userId }`
4. **Service key server-only** - Never import in client code
5. **Return 403 fast** - Don't process requests from unauthenticated users

## Tips

- Use the Grep tool to search for anti-patterns systematically
- Check browser Network tab - no direct database calls should appear
- Test with invalid token - should get 403
- Test as different user - should not see other users' data

## How to Verify

### After Remediation
- Run the audit again - all checks should pass
- Test with invalid token - should get 403
- Test with valid token but wrong userId - should get empty data or 403
- Check browser Network tab - no direct Supabase/database calls from frontend

### Production Checklist
- [ ] RLS enabled on all tables (as safety net)
- [ ] No permissive RLS policies
- [ ] All API routes have auth checks
- [ ] Service role key in server-only environment
- [ ] No database credentials in NEXT_PUBLIC_ variables
