---
name: frontend-security-audit
description: Audits frontend code for security vulnerabilities. Trigger words - frontend security, xss, csp, client security, localStorage security, dependency audit, npm audit
---

# Frontend Security Audit

Reviews frontend code for XSS vulnerabilities, insecure data storage, dependency vulnerabilities, and Content Security Policy configuration.

## When to Use This Skill

- User asks to "audit frontend security" or "check for XSS"
- Before deploying to production
- After adding user-generated content features
- User mentions concerns about client-side security
- After adding new npm dependencies
- User asks "is my frontend secure?"

## Core Principles

1. **Never trust user input** - Sanitize before rendering
2. **No secrets in frontend** - Only public keys, never API secrets
3. **CSP headers** - Prevent inline script execution
4. **Minimal localStorage** - Never store sensitive data
5. **Audit dependencies** - Check for known vulnerabilities

## Instructions

### Step 1: Audit XSS Vulnerabilities

Search for dangerous patterns:

```bash
# dangerouslySetInnerHTML (React)
grep -rn "dangerouslySetInnerHTML" --include="*.tsx" --include="*.jsx"

# innerHTML direct assignment
grep -rn "\.innerHTML\s*=" --include="*.ts" --include="*.tsx"

# document.write
grep -rn "document.write" --include="*.ts" --include="*.tsx"

# eval and Function constructor
grep -rn "eval\(|new Function\(" --include="*.ts" --include="*.tsx"

# Unescaped URL parameters in links
grep -rn "href=.*\${" --include="*.tsx"
```

**If dangerouslySetInnerHTML is used:**
- Is the content sanitized with DOMPurify?
- Is it from a trusted source (not user input)?

**Good:**
```typescript
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(content) }} />
```

**Bad:**
```typescript
<div dangerouslySetInnerHTML={{ __html: userInput }} /> // XSS!
```

### Step 2: Audit Client-Side Data Storage

Search for sensitive data in storage:

```bash
# localStorage usage
grep -rn "localStorage\." --include="*.ts" --include="*.tsx"

# sessionStorage usage
grep -rn "sessionStorage\." --include="*.ts" --include="*.tsx"

# IndexedDB usage
grep -rn "indexedDB\|openDB" --include="*.ts" --include="*.tsx"
```

**Never store:**
- Auth tokens (use httpOnly cookies)
- Passwords or secrets
- Personal data (PII)
- Payment information

**OK to store:**
- User preferences (theme, language)
- Non-sensitive UI state
- Public data caches

### Step 3: Audit Exposed Secrets

Search for hardcoded secrets:

```bash
# API keys in code
grep -rn "api[_-]?key\|apiKey" --include="*.ts" --include="*.tsx" --include="*.env*"

# Hardcoded secrets
grep -rn "secret\|password\|token" --include="*.ts" --include="*.tsx" | grep -v "\.d\.ts"

# Check environment variables
grep -rn "process\.env\." --include="*.ts" --include="*.tsx"
```

**Safe to expose (NEXT_PUBLIC_, VITE_):**
- Supabase anon key (if RLS configured)
- Stripe publishable key (pk_)
- Public API endpoints

**Never expose:**
- Service role keys
- Database passwords
- Private API keys (sk_)
- JWT secrets

### Step 4: Audit Dependencies

Run vulnerability checks:

```bash
# npm/pnpm/yarn audit
npm audit
# or
pnpm audit
# or
bun pm trust --all && bun audit

# Check for outdated packages
npm outdated
```

**Action required for:**
- Critical/High severity vulnerabilities
- Vulnerabilities in production dependencies
- Packages with no patches available

### Step 5: Audit Content Security Policy

Check for CSP headers:

```bash
# Next.js headers config
grep -rn "Content-Security-Policy" --include="*.config.*" --include="*.ts"

# Vercel.json or similar
grep -rn "Content-Security-Policy" vercel.json netlify.toml

# Meta tag CSP
grep -rn "http-equiv.*Content-Security-Policy" --include="*.tsx" --include="*.html"
```

**Recommended CSP:**
```
default-src 'self';
script-src 'self' 'unsafe-inline' 'unsafe-eval';  // Tighten in production
style-src 'self' 'unsafe-inline';
img-src 'self' data: https:;
font-src 'self';
connect-src 'self' https://api.yourservice.com;
```

### Step 6: Audit Third-Party Scripts

Search for external scripts:

```bash
# Script tags with external sources
grep -rn "<script.*src=" --include="*.tsx" --include="*.html"

# Dynamic script loading
grep -rn "createElement.*script" --include="*.ts" --include="*.tsx"
```

**For each external script:**
- Is it from a trusted CDN?
- Does it have integrity hash (SRI)?
- Is it necessary?

**Good:**
```html
<script
  src="https://cdn.example.com/lib.js"
  integrity="sha384-abc123..."
  crossorigin="anonymous"
/>
```

### Step 7: Audit URL Handling

Search for URL-based vulnerabilities:

```bash
# window.location manipulation
grep -rn "window\.location\s*=" --include="*.ts" --include="*.tsx"

# Redirect based on user input
grep -rn "redirect\|router\.push" --include="*.ts" --include="*.tsx"

# URL construction with user input
grep -rn "new URL\|encodeURI" --include="*.ts" --include="*.tsx"
```

**Open redirect risk:**
```typescript
// BAD - attacker can redirect to malicious site
router.push(searchParams.get('redirect'));

// GOOD - validate redirect URL
const redirect = searchParams.get('redirect');
if (redirect?.startsWith('/')) {
  router.push(redirect);
}
```

### Step 8: Audit Form Security

Check forms for security:

```bash
# Forms without proper attributes
grep -rn "<form" --include="*.tsx"

# Autocomplete on sensitive fields
grep -rn "type.*password\|type.*credit" --include="*.tsx"
```

**Required:**
- `autocomplete="off"` on sensitive fields
- `rel="noopener noreferrer"` on external links
- Proper input types (email, tel, password)

### Step 9: Audit Postmessage Usage

If using postMessage:

```bash
grep -rn "postMessage\|addEventListener.*message" --include="*.ts" --include="*.tsx"
```

**Required:**
- Validate origin before processing messages
- Specify target origin, never use `*`

```typescript
// GOOD
window.addEventListener('message', (event) => {
  if (event.origin !== 'https://trusted-site.com') return;
  // Process message
});

// BAD
window.addEventListener('message', (event) => {
  // Processing without origin check!
});
```

### Step 10: Generate Report

```markdown
## Frontend Security Audit Report

### XSS Prevention
- [ ] No unescaped dangerouslySetInnerHTML: [PASS/FAIL]
- [ ] No innerHTML with user input: [PASS/FAIL]
- [ ] No eval() or new Function(): [PASS/FAIL]
- [ ] User content sanitized with DOMPurify: [PASS/FAIL]

### Data Storage
- [ ] No auth tokens in localStorage: [PASS/FAIL]
- [ ] No sensitive data in client storage: [PASS/FAIL]
- [ ] Only preferences/UI state stored: [PASS/FAIL]

### Secrets Management
- [ ] No API secrets in frontend code: [PASS/FAIL]
- [ ] Only public keys exposed: [PASS/FAIL]
- [ ] Environment variables properly prefixed: [PASS/FAIL]

### Dependencies
- [ ] No critical vulnerabilities: [PASS/FAIL]
- [ ] No high vulnerabilities: [PASS/FAIL]
- [ ] Dependencies up to date: [PASS/FAIL]

### Content Security Policy
- [ ] CSP headers configured: [PASS/FAIL]
- [ ] Minimal script-src permissions: [PASS/FAIL]
- [ ] Third-party scripts have SRI: [PASS/FAIL]

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

### Stored XSS
**Risk:** Attacker stores malicious script, executes for other users
**Fix:** Sanitize with DOMPurify before rendering user content

### DOM-based XSS
**Risk:** URL parameters executed as code
**Fix:** Never use eval, sanitize URL params before use

### Sensitive Data Exposure
**Risk:** Tokens stolen via XSS, browser extensions, shared computers
**Fix:** httpOnly cookies, no localStorage for auth

### Dependency Vulnerabilities
**Risk:** Known exploits in outdated packages
**Fix:** Regular audits, automated updates (Dependabot)

### Clickjacking
**Risk:** Site embedded in iframe, user tricked into clicking
**Fix:** X-Frame-Options header, CSP frame-ancestors

## XSS Prevention Checklist

| Scenario | Safe Approach |
|----------|---------------|
| Displaying user text | React auto-escapes by default |
| Rendering HTML content | DOMPurify.sanitize() |
| Building URLs | encodeURIComponent() |
| Handling JSON | JSON.parse() (never eval) |
| External links | rel="noopener noreferrer" |

## Tips

- React auto-escapes text by default - only watch for dangerouslySetInnerHTML
- Check browser DevTools > Application > Storage for stored data
- Use `npm audit` regularly, integrate into CI
- CSP report-only mode helps identify issues before blocking
- Browser extensions can steal localStorage - don't store secrets

## How to Verify

### After Remediation
- `npm audit` returns no critical/high vulnerabilities
- DevTools > Application shows no sensitive data in storage
- Try XSS payload in inputs - should be escaped/blocked
- Check Network tab - no secret keys in requests
- CSP header visible in response headers

### Manual Testing
1. Enter `<script>alert('xss')</script>` in text fields - should display as text
2. Check localStorage in DevTools - no tokens or passwords
3. View page source - no hardcoded API keys
4. Run `npm audit` - no critical vulnerabilities
5. Check response headers - CSP present
