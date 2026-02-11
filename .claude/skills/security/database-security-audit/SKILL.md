---
name: database-security-audit
description: Audits database security architecture and access patterns. Trigger words - security audit, database security, rls audit, access control, security review, production ready
---

# Database Security Audit

Reviews database security configuration to ensure data is only accessed through properly authenticated backend routes - not directly from frontend code.

## Core Principle

There are two valid security models for Supabase projects. The right choice depends on your architecture. **For a detailed comparison with examples, migration paths, and decision matrix, see the Supabase skill's "Security Architecture: Choosing Your Approach" section.**

### Model 1: Database-Layer Security (RLS Policies)

For simple apps where the frontend queries the database directly using the anon key. RLS policies enforce permissions at the database level.

**When auditing a Model 1 project, verify:**
- RLS is enabled on ALL tables
- RLS policies exist for every table (no policies = data is inaccessible or wide open)
- Policies use `(SELECT auth.uid())` for performance
- Anon key is the only key exposed to the frontend
- Service role key is never in client code

### Model 2: Application-Layer Security (Middleware Auth) - Recommended for server-rendered apps

All data access goes through backend API routes or server actions. The frontend never directly queries the database.

**When auditing a Model 2 project, verify:**
1. All data access goes through API routes or server actions
2. Backend validates user authentication before any database operation
3. Invalid tokens return 403 immediately
4. Frontend NEVER directly talks to the database
5. RLS is enabled on all tables with NO permissive rules (defense in depth)
6. Only the service role key can access data (server-side only)

**Benefits of Model 2:**
- Ultimate control over access
- Can update logic without app releases (critical for mobile)
- Can cut access immediately if issues arise
- Fix bugs without redeploying clients

### Understanding "RLS enabled but no policies" Warnings

| Architecture | Is this a problem? | Action |
|---|---|---|
| Frontend queries DB directly (Model 1) | Yes - security risk | Implement RLS policies immediately |
| Backend API routes + middleware (Model 2) | No - expected behavior | Ignore, this is defense in depth |
| Exposing anon key to frontend | Yes - security risk | Add RLS policies or switch to Model 2 |

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

### Step 2: Determine Security Model

**Critical Question:** Which security model does this project use?

Search for frontend database access patterns:

```bash
# Check for frontend-to-database access
grep -rn "createBrowserClient" --include="*.tsx" --include="*.ts" components/ app/
grep -rn "\.from\(" --include="*.tsx" --include="*.ts" components/
grep -rn "\.select\(" --include="*.tsx" --include="*.ts" components/
grep -rn "\.insert\(" --include="*.tsx" --include="*.ts" components/
```

**If frontend queries the DB directly (Model 1):**
- This is valid IF RLS policies exist on every table
- Verify `.from()` calls in client components are protected by RLS
- Check that every table has appropriate SELECT/INSERT/UPDATE/DELETE policies
- Flag any table without policies as a critical issue

**If all DB access is through backend (Model 2):**
- All `.from()` calls should be in `app/api/` routes, server actions (`'use server'`), or server-only files
- `.from()` calls in `components/` or `'use client'` files are red flags
- RLS without policies is expected and correct (defense in depth)

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

1. **Identify the security model first** - Model 1 (RLS) and Model 2 (middleware) have different audit criteria
2. **Model 1: RLS policies are mandatory** - Every table needs policies, no exceptions
3. **Model 2: Auth middleware is mandatory** - Every API route and server action must check auth first
4. **Filter by userId** - Whether in RLS policies or backend code, always scope data to the user
5. **Service key server-only** - Never import in client code
6. **Return 403 fast** - Don't process requests from unauthenticated users

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
