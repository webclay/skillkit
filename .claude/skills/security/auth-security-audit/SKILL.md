---
name: auth-security-audit
description: Audits authentication and session security. Trigger words - auth security, session security, token storage, login security, oauth audit, csrf, authentication review
---

# Auth Security Audit

Reviews authentication implementation for secure session management, token handling, and protection against common auth vulnerabilities.

## When to Use This Skill

- User asks to "audit auth security" or "review authentication"
- Before deploying to production
- After setting up Better Auth, NextAuth, or custom auth
- User mentions concerns about session security
- After implementing OAuth providers
- User asks "is my login secure?"

## Core Principles

1. **Tokens in httpOnly cookies** - Never localStorage for auth tokens
2. **Session rotation** - New session ID after login
3. **CSRF protection** - State tokens on all mutations
4. **Secure password reset** - Time-limited, single-use tokens
5. **OAuth state validation** - Prevent CSRF on OAuth flows

## Instructions

### Step 1: Identify Auth System

Check `package.json` for:
- `better-auth` - Better Auth
- `next-auth` or `@auth/core` - Auth.js
- `@supabase/auth-helpers` - Supabase Auth
- Custom JWT implementation

### Step 2: Audit Token Storage

**Critical:** Where are auth tokens stored?

Search for dangerous patterns:

```bash
# localStorage token storage (BAD)
grep -rn "localStorage.setItem.*token" --include="*.ts" --include="*.tsx"
grep -rn "localStorage.setItem.*session" --include="*.ts" --include="*.tsx"

# sessionStorage (also bad for auth)
grep -rn "sessionStorage.setItem.*token" --include="*.ts" --include="*.tsx"
```

**Good:** Tokens in httpOnly cookies (set by server)
**Bad:** Tokens in localStorage, sessionStorage, or JS-accessible cookies

### Step 3: Audit Session Configuration

Check session settings:

```typescript
// Better Auth - check config
// Look for session configuration
grep -rn "session:" --include="auth.ts" --include="auth.config.ts"
```

**Required settings:**
- `httpOnly: true` - Cookie not accessible via JavaScript
- `secure: true` - Cookie only sent over HTTPS (production)
- `sameSite: 'lax'` or `'strict'` - CSRF protection
- Reasonable expiration (not infinite)

### Step 4: Audit CSRF Protection

Check for CSRF tokens on mutations:

```bash
# Forms without CSRF tokens
grep -rn "<form" --include="*.tsx" | grep -v "csrf"

# POST/PUT/DELETE without CSRF validation
grep -rn "method.*POST\|PUT\|DELETE" --include="*.ts" --include="*.tsx"
```

**Better Auth:** Uses built-in CSRF protection
**Custom forms:** Must include CSRF token

### Step 5: Audit Password Reset Flow

Search for password reset implementation:

```bash
grep -rn "reset.*password\|forgot.*password" --include="*.ts" --include="*.tsx"
grep -rn "passwordReset\|password-reset" --include="*.ts" --include="*.tsx"
```

**Required:**
- Token expires (15-60 minutes max)
- Token is single-use (invalidated after use)
- Token is cryptographically random (not predictable)
- Rate limiting on reset requests
- No user enumeration (same response for valid/invalid email)

### Step 6: Audit OAuth Implementation

If OAuth is used:

```bash
grep -rn "oauth\|OAuth\|google\|github\|discord" --include="*.ts" --include="*.tsx"
```

**Required:**
- State parameter validated on callback
- PKCE for public clients (SPAs, mobile)
- Redirect URI validation (exact match, no wildcards)
- Token exchange happens server-side

### Step 7: Audit Session Lifecycle

Check for proper session handling:

```bash
# Session creation
grep -rn "createSession\|signIn\|login" --include="*.ts"

# Session destruction
grep -rn "deleteSession\|signOut\|logout" --include="*.ts"
```

**Required:**
- New session ID generated on login (prevents session fixation)
- All sessions invalidated on password change
- Logout actually destroys server-side session
- Session timeout implemented

### Step 8: Audit Rate Limiting

Check for brute force protection:

```bash
grep -rn "rateLimit\|rate-limit\|throttle" --include="*.ts"
```

**Required on:**
- Login endpoint
- Password reset request
- Account creation
- Any endpoint accepting passwords

### Step 9: Generate Report

```markdown
## Auth Security Audit Report

### Token Storage
- [ ] Tokens in httpOnly cookies: [PASS/FAIL]
- [ ] No tokens in localStorage: [PASS/FAIL]
- [ ] No tokens in sessionStorage: [PASS/FAIL]

### Session Security
- [ ] httpOnly flag set: [PASS/FAIL]
- [ ] Secure flag set (production): [PASS/FAIL]
- [ ] SameSite attribute set: [PASS/FAIL]
- [ ] Session rotation on login: [PASS/FAIL]
- [ ] Session destroyed on logout: [PASS/FAIL]

### CSRF Protection
- [ ] CSRF tokens on forms: [PASS/FAIL]
- [ ] State parameter on OAuth: [PASS/FAIL]

### Password Security
- [ ] Reset tokens expire: [PASS/FAIL]
- [ ] Reset tokens single-use: [PASS/FAIL]
- [ ] No user enumeration: [PASS/FAIL]
- [ ] Rate limiting on auth endpoints: [PASS/FAIL]

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

### Session Fixation
**Risk:** Attacker sets session ID before login, then hijacks after
**Fix:** Generate new session ID on successful login

### Token in localStorage
**Risk:** XSS can steal tokens, full account takeover
**Fix:** Use httpOnly cookies, tokens never touch JavaScript

### Missing CSRF Protection
**Risk:** Malicious site can perform actions as logged-in user
**Fix:** CSRF tokens on all state-changing requests

### Predictable Reset Tokens
**Risk:** Attacker can guess reset token, take over account
**Fix:** Use cryptographically secure random tokens (32+ bytes)

### No Rate Limiting
**Risk:** Brute force attacks on login/reset
**Fix:** Implement rate limiting (e.g., 5 attempts per minute)

## Better Auth Specific Checks

Better Auth handles many security concerns by default. Verify:

```typescript
// auth.ts - check these are NOT disabled
export const auth = betterAuth({
  session: {
    // Should NOT have these insecure settings:
    // cookieOptions: { httpOnly: false }  // BAD
    // cookieOptions: { secure: false }     // BAD in production
  }
})
```

Check middleware protects routes:

```typescript
// middleware.ts or route beforeLoad
// Verify session check happens before protected routes
```

## Tips

- Use the Grep tool to search for patterns systematically
- Check browser DevTools > Application > Cookies for cookie flags
- Test logout - does it actually clear the session?
- Try accessing protected routes after logout
- Check if password change invalidates other sessions

## How to Verify

### After Remediation
- Tokens not visible in browser DevTools > Application > Local Storage
- Cookies show httpOnly, Secure, SameSite flags
- Login creates new session ID (check Set-Cookie header)
- Logout clears all auth cookies
- Password reset token works once, then fails
- Rate limiting blocks after threshold (test with curl)

### Manual Testing
1. Login, check cookies have httpOnly flag
2. Try XSS payload `<script>alert(document.cookie)</script>` - auth cookies should NOT appear
3. Logout, try to access protected route - should redirect
4. Request password reset, wait 1 hour, try to use - should fail
5. Try 10 rapid login attempts - should be rate limited
