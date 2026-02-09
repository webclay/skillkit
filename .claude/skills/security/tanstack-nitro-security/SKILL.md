---
name: tanstack-nitro-security
description: Implements Nitro middleware security headers for TanStack Start projects. Trigger words - tanstack start security, nitro security headers, tanstack security middleware, secure tanstack start, production tanstack
---

# TanStack Start - Nitro Security Headers

Implements server-side security headers using Nitro middleware to protect TanStack Start applications against clickjacking, MIME-type sniffing, XSS attacks, and enforce HTTPS.

## When to Use This Skill

- Project uses TanStack Start (has `@tanstack/react-start`)
- Setting up new TanStack Start project
- Preparing TanStack Start app for production
- User asks about securing TanStack Start
- User asks about security headers in TanStack Start
- During deployment security audit of TanStack Start project

## Prerequisites

This skill requires:
- TanStack Start project (check `package.json` for `@tanstack/react-start`)
- Nitro (comes with TanStack Start)
- Vite configuration already set up

## Instructions

### Step 1: Verify and Update Nitro Dependency

Check `package.json` for the Nitro dependency. If it uses nightly channel, update to stable version.

**Check current version:**
```bash
grep '"nitro"' package.json
```

**If it shows `"nitro": "npm:nitro-nightly@latest"`, replace with:**
```json
"nitro": "^3.0.0"
```

**If already stable version (e.g., `"nitro": "^3.0.0"`), skip this step.**

After updating, run:
```bash
bun install
```

### Step 2: Add Nitro Vite Plugin

Open `vite.config.ts` and ensure the Nitro plugin is properly configured.

**Add import at top of file:**
```typescript
import { nitro } from "nitro/vite";
```

**Add `nitro()` to plugins array** - must come after `tanstackStart()`:
```typescript
plugins: [
  tanstackStart({
    srcDirectory: "src",  // or "app" depending on project
  }),
  nitro(),  // <-- MUST come after tanstackStart()
  // ... rest of plugins
],
```

**Critical:** The order matters. `nitro()` must come after `tanstackStart()` for proper environment variable handling.

### Step 3: Ensure nitro.config.ts Exists

Check if `nitro.config.ts` exists at project root. This file is **required** for Nitro to work properly.

**If missing, create `nitro.config.ts`:**
```typescript
import { defineNitroConfig } from "nitro/config";

export default defineNitroConfig({});
```

Even an empty config is fine - the file just needs to exist.

### Step 4: Create Security Headers Middleware

Create the directory structure and middleware file:

```bash
mkdir -p server/middleware
```

**Create `server/middleware/security-headers.ts`:**
```typescript
import { defineEventHandler, setHeaders } from "h3";

export default defineEventHandler((event) => {
  const headers: Record<string, string> = {
    "X-Content-Type-Options": "nosniff",
    "X-Frame-Options": "DENY",
    "Referrer-Policy": "strict-origin-when-cross-origin",
    "Permissions-Policy":
      "camera=(), microphone=(), geolocation=(), payment=()",
    "Content-Security-Policy":
      "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self'; frame-ancestors 'none';",
  };

  // Only add HSTS in production to avoid breaking localhost
  if (process.env.NODE_ENV === "production") {
    headers["Strict-Transport-Security"] =
      "max-age=31536000; includeSubDomains";
  }

  setHeaders(event, headers);
});
```

**How it works:**
- Nitro auto-discovers files in `server/middleware/`
- Runs on every HTTP request
- `h3` library is included with Nitro (no install needed)
- No registration required

**What each header does:**

| Header | Purpose | Value |
|--------|---------|-------|
| `X-Content-Type-Options` | Prevents MIME-type sniffing | `nosniff` |
| `X-Frame-Options` | Prevents clickjacking | `DENY` |
| `Referrer-Policy` | Controls referer information | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | Restricts browser features | Disables camera, mic, location, payment |
| `Content-Security-Policy` | Protects against XSS and injection attacks | Restricts resource loading |
| `Strict-Transport-Security` | Forces HTTPS (production only) | 1 year max-age, includes subdomains |

**Important notes:**
- `X-XSS-Protection` header is **not included** because it's deprecated and can introduce vulnerabilities in older browsers
- Modern browsers rely on Content-Security-Policy instead
- HSTS only applies in production to avoid breaking localhost during development

### Step 5: Add .nitro/ to .gitignore

Nitro creates a build cache directory that should not be committed.

**Add to `.gitignore`:**
```
/.nitro
```

### Step 6: Verify Implementation

**Start dev server:**
```bash
bun run dev
```

**Test in browser:**
1. Open the application in browser
2. Open DevTools (F12)
3. Go to Network tab
4. Refresh page and click any request
5. Check Response Headers section

**Expected headers in development:**
- `x-content-type-options: nosniff`
- `x-frame-options: DENY`
- `referrer-policy: strict-origin-when-cross-origin`
- `permissions-policy: camera=(), microphone=(), geolocation=(), payment=()`
- `content-security-policy: default-src 'self'; script-src...`

**Expected headers in production (adds HSTS):**
- All of the above, plus:
- `strict-transport-security: max-age=31536000; includeSubDomains`

**Note:** HSTS is production-only to avoid breaking localhost in development.

## Common Issues

### Headers Not Appearing

**Symptom:** Security headers missing in Network tab

**Causes:**
1. Nitro plugin not in `vite.config.ts`
2. Wrong plugin order (nitro before tanstackStart)
3. `nitro.config.ts` missing
4. Middleware file in wrong location

**Fix:**
- Verify `nitro()` plugin exists and comes after `tanstackStart()`
- Ensure `nitro.config.ts` exists at project root
- Check middleware path: `server/middleware/security-headers.ts`
- Restart dev server

### Environment Variables Not Loading

**Symptom:** Server code can't access env vars

**Cause:** Missing `nitro.config.ts` or wrong plugin order

**Fix:**
- Create `nitro.config.ts` at project root
- Verify plugin order in `vite.config.ts`

### Build Errors

**Symptom:** `npm run build` fails with Nitro errors

**Cause:** Conflicting Nitro versions or config issues

**Fix:**
- Ensure stable Nitro version: `"nitro": "^3.0.0"`
- Run `bun install` again
- Delete `.nitro/` and `node_modules/`, reinstall

## Security Best Practices

### Production Checklist

Before deploying TanStack Start to production:

- [ ] Nitro security headers middleware active
- [ ] HTTPS enforced (hosting provider setting)
- [ ] All secrets in environment variables
- [ ] `.env` files in `.gitignore`
- [ ] Custom error pages (no stack traces)
- [ ] Console.logs removed
- [ ] Database connections use SSL

### Content Security Policy Customization

The default CSP is permissive to work with most TanStack Start apps. You may need to adjust it based on your app's needs:

**If using external APIs or CDNs:**
```typescript
"Content-Security-Policy":
  "default-src 'self'; " +
  "script-src 'self' 'unsafe-inline'; " +
  "style-src 'self' 'unsafe-inline'; " +
  "img-src 'self' data: https:; " +
  "font-src 'self' data:; " +
  "connect-src 'self' https://api.example.com; " + // Add your APIs
  "frame-ancestors 'none';",
```

**If not using inline styles/scripts (most secure):**
```typescript
"Content-Security-Policy":
  "default-src 'self'; " +
  "script-src 'self'; " +  // Remove 'unsafe-inline'
  "style-src 'self'; " +   // Remove 'unsafe-inline'
  "img-src 'self' data: https:; " +
  "font-src 'self' data:; " +
  "connect-src 'self'; " +
  "frame-ancestors 'none';",
```

**CSP Violations:** Check browser console for CSP violations during development. Adjust the policy to allow legitimate resources while blocking unsafe ones.

### Additional Security Layers

This middleware provides foundational security headers. Consider adding:

1. **Rate Limiting**
   - Implement in API routes
   - Protect against brute force

2. **Input Validation**
   - Validate all user input
   - Use Zod schemas

3. **Authentication Security**
   - Use Better Auth skill patterns
   - Secure session management

## Customizing Headers

### Allow Embedding in Iframes

If you need to allow embedding (e.g., in Notion, Confluence):

```typescript
"X-Frame-Options": "SAMEORIGIN",  // Change from DENY
```

### Enable Specific Permissions

To allow camera or location access:

```typescript
"Permissions-Policy": "camera=(self), geolocation=(self)",
```

### Add CSP for External Resources

If your app loads resources from external domains:

```typescript
"Content-Security-Policy":
  "default-src 'self'; " +
  "script-src 'self' 'unsafe-inline' https://cdn.example.com; " +
  "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; " +
  "img-src 'self' data: https: blob:; " +
  "font-src 'self' data: https://fonts.gstatic.com; " +
  "connect-src 'self' https://api.example.com wss://realtime.example.com; " +
  "frame-ancestors 'none';",
```

## Files Changed Summary

| File | Action | Purpose |
|------|--------|---------|
| `package.json` | Update (if needed) | Ensure stable Nitro version |
| `vite.config.ts` | Add import and plugin | Enable Nitro middleware |
| `nitro.config.ts` | Create | Required for Nitro |
| `server/middleware/security-headers.ts` | Create | Security headers logic |
| `.gitignore` | Add `/.nitro` | Exclude build cache |

**No changes needed to:**
- Routes
- Components
- Existing configuration
- API endpoints

## Tips

- Headers apply to all routes automatically
- No performance impact (middleware is fast)
- Works in dev and production
- Can add multiple middleware files
- Middleware runs in alphabetical order
- Use conditional headers if needed (check `event.path`)

## How to Verify

### Quick Test
```bash
# Start dev server
bun run dev

# Test with curl (in another terminal)
curl -I http://localhost:3000

# Should see security headers in response (HSTS only in production)
```

### Production Test
After deploying:
1. Visit your production URL
2. Check headers in DevTools
3. Run security audit: https://securityheaders.com
4. Should receive grade A or better

### Automated Testing
Add to test suite:
```typescript
test('security headers present', async () => {
  const response = await fetch('http://localhost:3000');
  expect(response.headers.get('x-frame-options')).toBe('DENY');
  expect(response.headers.get('x-content-type-options')).toBe('nosniff');
  expect(response.headers.get('content-security-policy')).toBeTruthy();
  expect(response.headers.get('x-xss-protection')).toBeNull(); // Should NOT be present
  // In production, also check HSTS
});
```

## Reference

- [Nitro Documentation](https://nitro.unjs.io/)
- [H3 Event Handler API](https://h3.unjs.io/)
- [OWASP Security Headers](https://owasp.org/www-project-secure-headers/)
- [TanStack Start Skill](../framework/tanstack-start/SKILL.md)
- [Deployment Security Audit](../security/deployment-security-audit/SKILL.md)
