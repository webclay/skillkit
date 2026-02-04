# Database Security Audit Checklists

Use these checklists based on your database setup.

## Universal Checklist (All Databases)

### Architecture
- [ ] All database access goes through backend API routes or server actions
- [ ] Frontend components never import database clients directly
- [ ] No `.from()`, `.select()`, `.insert()` calls in client components
- [ ] Database credentials not in any `NEXT_PUBLIC_*` or `VITE_*` variables

### Authentication
- [ ] Every API route checks auth before processing
- [ ] Every server action checks auth before processing
- [ ] Auth check happens BEFORE any other processing
- [ ] Invalid/expired tokens return 403 immediately
- [ ] Token refresh is handled properly

### Authorization
- [ ] Every query filters by userId or organizationId
- [ ] Delete operations verify ownership
- [ ] Update operations verify ownership
- [ ] No queries return data user shouldn't see

### Environment
- [ ] `.env` file is in `.gitignore`
- [ ] Production uses different credentials than development
- [ ] Service/admin keys only in server environment
- [ ] No secrets in client bundle (check build output)

---

## Supabase-Specific Checklist

### RLS Configuration
- [ ] RLS is ENABLED on all tables
- [ ] NO permissive policies (or minimal, carefully audited ones)
- [ ] `anon` role cannot SELECT, INSERT, UPDATE, DELETE sensitive data
- [ ] Only `service_role` has full access
- [ ] Tables with public data have explicit, minimal policies

### Client Configuration
- [ ] `NEXT_PUBLIC_SUPABASE_ANON_KEY` only used for auth flow
- [ ] `SUPABASE_SERVICE_ROLE_KEY` only in server-side code
- [ ] Server client uses service role key
- [ ] Browser client only used for auth, not data

### Edge Functions / API Routes
- [ ] All data mutations go through Edge Functions or API routes
- [ ] Edge Functions validate JWT before database access
- [ ] Edge Functions use service role client
- [ ] CORS configured properly on Edge Functions

### Auth Flow
- [ ] Using `supabase.auth.getUser()` not just `getSession()`
- [ ] Session refresh handled in middleware
- [ ] Logout clears all session data

---

## Prisma-Specific Checklist

### Connection Security
- [ ] `DATABASE_URL` only in server environment
- [ ] Connection string uses SSL (`?sslmode=require`)
- [ ] Connection pooling configured for serverless
- [ ] Prisma client is singleton (not recreated per request)

### Query Security
- [ ] All queries include `where: { userId }` clause
- [ ] `findMany` has proper limits to prevent data dump
- [ ] No raw SQL without parameterization
- [ ] Sensitive fields excluded from default select

### Schema Security
- [ ] User ID is required foreign key on owned tables
- [ ] Cascade deletes won't expose orphaned data
- [ ] Sensitive data fields properly typed
- [ ] `@@index` on userId for performance

---

## Drizzle-Specific Checklist

### Connection Security
- [ ] Database URL only in server environment
- [ ] Using connection pooling for edge deployment
- [ ] SSL enabled on connection

### Query Security
- [ ] Using Drizzle's `eq()` operators (not string interpolation)
- [ ] All queries filter by userId
- [ ] Using `sql` template tag for raw queries (parameterized)
- [ ] Transactions used for multi-step operations

---

## Better Auth Checklist

### Session Validation
- [ ] Using `auth.api.getSession({ headers: request.headers })` in API routes
- [ ] Using `auth.api.getSession({ headers: await headers() })` in server actions
- [ ] Session check is FIRST thing in every protected route
- [ ] 403 returned immediately for invalid sessions

### Token Security
- [ ] `BETTER_AUTH_SECRET` is strong (32+ characters)
- [ ] `BETTER_AUTH_SECRET` not exposed to client
- [ ] Session expiration configured appropriately
- [ ] Refresh tokens working correctly

### User Data
- [ ] User ID from session used for all queries (not from request body)
- [ ] `session.user.id` is the source of truth
- [ ] No trusting client-provided userId

---

## API Security Checklist

### Input Validation
- [ ] All inputs validated with Zod or similar
- [ ] Validation happens after auth, before database
- [ ] Error messages don't leak sensitive info
- [ ] File uploads have size and type limits

### Response Security
- [ ] Sensitive fields stripped from responses
- [ ] Error responses are generic (not stack traces)
- [ ] Proper status codes (401 vs 403 vs 404)
- [ ] Rate limiting on sensitive endpoints

### Headers
- [ ] CORS configured restrictively
- [ ] No `Access-Control-Allow-Origin: *` in production
- [ ] Security headers set (CSP, X-Frame-Options, etc.)

---

## Production Deployment Checklist

### Environment
- [ ] All secrets in secure environment variables
- [ ] No `.env` file deployed
- [ ] Different database for production
- [ ] Logging doesn't include sensitive data

### Access Control
- [ ] Can revoke access quickly if needed
- [ ] Have incident response plan
- [ ] Know how to rotate compromised keys

### Monitoring
- [ ] Failed auth attempts are logged
- [ ] Unusual query patterns would be noticed
- [ ] Database access is auditable

---

## Quick Audit Commands

```bash
# 1. Check for frontend database access
grep -rn "\.from\(" --include="*.tsx" components/ app/

# 2. Find API routes without auth
find app/api -name "route.ts" -exec grep -L "getSession" {} \;

# 3. Check for exposed secrets
grep -rn "SERVICE" --include="*.ts" lib/ app/

# 4. Find RLS policies
grep -rn "CREATE POLICY" migrations/ supabase/

# 5. Check env files for client-exposed secrets
grep "NEXT_PUBLIC_" .env* | grep -v "URL\|ANON"
```
