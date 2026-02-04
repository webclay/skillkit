---
name: api-security-audit
description: Audits API endpoints for security vulnerabilities. Trigger words - api security, input validation, rate limiting, cors, injection, endpoint security, api audit
---

# API Security Audit

Reviews API endpoints for input validation, injection prevention, rate limiting, CORS configuration, and secure error handling.

## When to Use This Skill

- User asks to "audit API security" or "review endpoints"
- Before deploying to production
- After adding new API routes or server functions
- User mentions concerns about injection or validation
- After implementing tRPC, REST APIs, or server actions
- User asks "are my APIs secure?"

## Core Principles

1. **Validate all input** - Use Zod schemas on every endpoint
2. **Sanitize output** - Never reflect user input without encoding
3. **Rate limit everything** - Protect against abuse and DoS
4. **Proper CORS** - Don't use `*` in production
5. **Safe error messages** - Never leak stack traces or internals

## Instructions

### Step 1: Identify API Types

Check for:
- `app/api/` - Next.js/TanStack API routes
- `server/` or `trpc/` - tRPC routers
- `'use server'` - Server actions
- `functions/` - Serverless functions

### Step 2: Audit Input Validation

**Critical:** Every endpoint must validate input with Zod or similar.

Search for unvalidated input:

```bash
# API routes without validation
grep -rn "req.body\|req.query\|req.params" --include="*.ts" app/api/

# Server actions without validation
grep -rn "'use server'" -A 20 --include="*.ts" | grep -v "z\.\|schema\|parse"

# tRPC procedures without input validation
grep -rn "\.mutation\|\.query" --include="*.ts" | grep -v "input:"
```

**Good pattern:**
```typescript
const schema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
});

const data = schema.parse(req.body);
```

**Bad pattern:**
```typescript
const { email, name } = req.body; // No validation!
```

### Step 3: Audit SQL/NoSQL Injection

Search for string concatenation in queries:

```bash
# Raw SQL with string interpolation (DANGEROUS)
grep -rn "\`.*\${.*}\`" --include="*.ts" | grep -i "select\|insert\|update\|delete"

# Prisma raw queries
grep -rn "\$queryRaw\|\$executeRaw" --include="*.ts"

# Drizzle raw SQL
grep -rn "sql\`" --include="*.ts"
```

**Good:** Parameterized queries, ORM methods
**Bad:** String concatenation in SQL

### Step 4: Audit Rate Limiting

Check for rate limiting implementation:

```bash
grep -rn "rateLimit\|rateLimiter\|upstash.*ratelimit" --include="*.ts"
```

**Required on:**
- Authentication endpoints (login, register, reset)
- Resource-intensive operations (search, export)
- Public APIs
- File uploads

**Common solutions:**
- `@upstash/ratelimit` - Serverless-friendly
- `express-rate-limit` - Express middleware
- Vercel/Cloudflare built-in rate limiting

### Step 5: Audit CORS Configuration

Search for CORS settings:

```bash
grep -rn "cors\|Access-Control" --include="*.ts" --include="*.config.*"
grep -rn "allowedOrigins\|origin:" --include="*.ts"
```

**Good:**
```typescript
cors({
  origin: ['https://myapp.com', 'https://www.myapp.com'],
  credentials: true
})
```

**Bad:**
```typescript
cors({ origin: '*' })  // Allows any origin
cors({ origin: true }) // Reflects any origin
```

### Step 6: Audit Error Handling

Search for error responses:

```bash
grep -rn "catch\|error\|Error" --include="*.ts" app/api/ server/
```

**Good:**
```typescript
catch (error) {
  logger.error('Payment failed', { userId, error });
  return { error: 'Payment processing failed' }; // Generic message
}
```

**Bad:**
```typescript
catch (error) {
  return { error: error.message, stack: error.stack }; // Leaks internals!
}
```

### Step 7: Audit File Uploads

If file uploads exist:

```bash
grep -rn "upload\|formData\|multipart\|multer" --include="*.ts"
```

**Required:**
- File type validation (check magic bytes, not just extension)
- File size limits
- Virus scanning for user-uploaded files
- Store outside webroot or use signed URLs
- Randomize filenames (prevent path traversal)

### Step 8: Audit Sensitive Data Exposure

Check what's returned in responses:

```bash
# Returning full user objects
grep -rn "return.*user\|res.json.*user" --include="*.ts"

# Check for password/secret in responses
grep -rn "password\|secret\|token" --include="*.ts" app/api/
```

**Good:** Return only needed fields
```typescript
return { id: user.id, name: user.name, email: user.email };
```

**Bad:** Return entire object
```typescript
return user; // May include passwordHash, internal IDs, etc.
```

### Step 9: Audit Request Size Limits

Check for body size limits:

```bash
grep -rn "bodyParser\|limit.*mb\|limit.*kb" --include="*.ts" --include="*.config.*"
```

**Required:** Set reasonable limits to prevent DoS
```typescript
// next.config.js
api: {
  bodyParser: {
    sizeLimit: '1mb'
  }
}
```

### Step 10: Generate Report

```markdown
## API Security Audit Report

### Input Validation
- [ ] All endpoints use Zod/schema validation: [PASS/FAIL]
- [ ] No unvalidated req.body access: [PASS/FAIL]
- [ ] Server actions validate input: [PASS/FAIL]

### Injection Prevention
- [ ] No SQL string concatenation: [PASS/FAIL]
- [ ] Parameterized queries used: [PASS/FAIL]
- [ ] NoSQL injection prevented: [PASS/FAIL]

### Rate Limiting
- [ ] Auth endpoints rate limited: [PASS/FAIL]
- [ ] Public APIs rate limited: [PASS/FAIL]
- [ ] Resource-intensive operations limited: [PASS/FAIL]

### CORS Configuration
- [ ] Specific origins configured: [PASS/FAIL]
- [ ] No wildcard (*) in production: [PASS/FAIL]

### Error Handling
- [ ] Generic error messages to clients: [PASS/FAIL]
- [ ] No stack traces in responses: [PASS/FAIL]
- [ ] Errors logged server-side: [PASS/FAIL]

### Data Exposure
- [ ] Minimal data in responses: [PASS/FAIL]
- [ ] No passwords/secrets returned: [PASS/FAIL]
- [ ] Sensitive fields excluded: [PASS/FAIL]

### Critical Issues (must fix)
1. [Issue with file path and line number]
   - **Risk:** [What could go wrong]
   - **Fix:** [How to remediate]

### Warnings (should fix)
1. [Issue description]

### Recommendations
1. [Improvement suggestion]
```

## Common Vulnerabilities

### SQL Injection
**Risk:** Attacker executes arbitrary SQL, dumps/modifies database
**Fix:** Always use parameterized queries or ORM methods

### Mass Assignment
**Risk:** Attacker sets fields they shouldn't (isAdmin: true)
**Fix:** Explicit allow-list of fields, use Zod pick/omit

### Broken Rate Limiting
**Risk:** Brute force, credential stuffing, DoS
**Fix:** Rate limit by IP + user, use sliding window

### Verbose Errors
**Risk:** Stack traces reveal framework, versions, file paths
**Fix:** Log details server-side, return generic messages

### CORS Misconfiguration
**Risk:** Malicious sites can make authenticated requests
**Fix:** Explicit origin allowlist, no wildcards with credentials

## Zod Validation Patterns

```typescript
// API route validation
import { z } from 'zod';

const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(13).max(120).optional(),
});

export async function POST(req: Request) {
  const body = await req.json();

  const result = CreateUserSchema.safeParse(body);
  if (!result.success) {
    return Response.json(
      { error: 'Invalid input', details: result.error.flatten() },
      { status: 400 }
    );
  }

  const { email, name, age } = result.data;
  // Now safely use validated data
}
```

## tRPC Validation

```typescript
// tRPC automatically validates with Zod
export const userRouter = router({
  create: protectedProcedure
    .input(z.object({
      email: z.string().email(),
      name: z.string().min(1),
    }))
    .mutation(async ({ input, ctx }) => {
      // input is already validated
    }),
});
```

## Tips

- Use Grep tool to search systematically
- Check network tab - are responses minimal?
- Test with invalid input - proper error messages?
- Try SQL injection payloads: `'; DROP TABLE users; --`
- Check response headers for security headers

## How to Verify

### After Remediation
- Invalid input returns 400 with generic message
- SQL injection attempts fail safely
- 100 rapid requests trigger rate limit
- CORS blocks requests from unauthorized origins
- Error responses contain no stack traces

### Manual Testing
1. Send malformed JSON - should return 400, not 500
2. Send unexpected fields - should be ignored or rejected
3. Try `' OR '1'='1` in text fields - should not affect query
4. Make 100 requests in 10 seconds - should be rate limited
5. Check response for password fields - should never appear
