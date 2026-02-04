---
name: deployment-security-audit
description: Audits deployment and infrastructure security. Trigger words - deployment security, https, security headers, secrets management, production security, infrastructure audit, deploy audit
---

# Deployment Security Audit

Reviews deployment configuration for HTTPS, security headers, secrets management, logging, and production hardening.

## When to Use This Skill

- User asks to "audit deployment security" or "production ready check"
- Before deploying to production
- After setting up Vercel, Netlify, or other hosting
- User mentions concerns about infrastructure security
- Setting up CI/CD pipelines
- User asks "is my deployment secure?"

## Core Principles

1. **HTTPS everywhere** - All traffic encrypted, no HTTP
2. **Security headers** - Protect against common attacks
3. **Secrets in environment** - Never in code or git
4. **Minimal permissions** - Least privilege for all services
5. **Logging without secrets** - Audit trail, no sensitive data

## Instructions

### Step 1: Audit HTTPS Configuration

Check for HTTPS enforcement:

```bash
# Check for HTTP redirects
grep -rn "http://" --include="*.ts" --include="*.tsx" --include="*.env*" | grep -v localhost

# Check Vercel/Netlify config
cat vercel.json netlify.toml 2>/dev/null

# Check for forced HTTPS in headers
grep -rn "Strict-Transport-Security" --include="*.config.*" --include="*.ts"
```

**Required:**
- All production URLs use HTTPS
- HTTP redirects to HTTPS
- HSTS header configured

### Step 2: Audit Security Headers

Check for security headers configuration:

```bash
# Next.js headers
grep -rn "headers" next.config.* --include="*.js" --include="*.ts" -A 20

# Vercel.json headers
grep -rn "headers" vercel.json -A 20

# Manual header setting
grep -rn "setHeader\|X-Frame\|X-Content\|Strict-Transport" --include="*.ts"
```

**Required headers:**

| Header | Purpose | Recommended Value |
|--------|---------|-------------------|
| `Strict-Transport-Security` | Force HTTPS | `max-age=31536000; includeSubDomains` |
| `X-Frame-Options` | Prevent clickjacking | `DENY` or `SAMEORIGIN` |
| `X-Content-Type-Options` | Prevent MIME sniffing | `nosniff` |
| `Referrer-Policy` | Control referer info | `strict-origin-when-cross-origin` |
| `X-XSS-Protection` | Legacy XSS filter | `1; mode=block` |
| `Permissions-Policy` | Limit browser features | Based on needs |

**Example next.config.js:**
```javascript
async headers() {
  return [
    {
      source: '/:path*',
      headers: [
        { key: 'X-Frame-Options', value: 'DENY' },
        { key: 'X-Content-Type-Options', value: 'nosniff' },
        { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
        { key: 'Strict-Transport-Security', value: 'max-age=31536000; includeSubDomains' },
      ],
    },
  ];
}
```

### Step 3: Audit Secrets Management

Check for exposed secrets:

```bash
# Secrets in code
grep -rn "sk_live\|sk_test\|password\s*=\|secret\s*=" --include="*.ts" --include="*.tsx"

# .env files in git
git ls-files | grep -E "\.env$|\.env\.local$|\.env\.production$"

# Check .gitignore
grep -E "\.env|secrets" .gitignore
```

**Required:**
- All secrets in environment variables
- `.env*` files in `.gitignore`
- Production secrets in hosting provider (Vercel, etc.)
- No secrets committed to git history

**Check git history for leaked secrets:**
```bash
# Search git history for secrets (simplified)
git log -p | grep -E "sk_live|password|secret" | head -20
```

### Step 4: Audit Error Pages

Check production error handling:

```bash
# Custom error pages
ls -la app/error.tsx pages/_error.tsx app/not-found.tsx 2>/dev/null

# Error boundary implementations
grep -rn "ErrorBoundary\|error\.tsx" --include="*.tsx"
```

**Required:**
- Custom 404 page (no stack traces)
- Custom 500 page (no stack traces)
- Error boundaries catch client errors
- Generic messages to users

### Step 5: Audit Logging Configuration

Check logging setup:

```bash
# Console.log in production
grep -rn "console\.log\|console\.error" --include="*.ts" --include="*.tsx" | wc -l

# Structured logging
grep -rn "logger\.\|winston\|pino\|logLevel" --include="*.ts"

# Check for logged secrets
grep -rn "logger.*password\|console.*token\|log.*secret" --include="*.ts"
```

**Required:**
- Remove console.log in production (or use proper logger)
- Never log passwords, tokens, or PII
- Use structured logging (JSON format)
- Appropriate log levels (error, warn, info)

### Step 6: Audit Build Configuration

Check build settings:

```bash
# Source maps in production
grep -rn "sourceMap\|productionSourceMap" --include="*.config.*"

# Debug mode flags
grep -rn "DEBUG\|NODE_ENV" --include="*.ts" --include="*.config.*"

# Check for development dependencies in production
grep -rn "devDependencies" package.json -A 50
```

**Required:**
- Source maps disabled in production (or uploaded privately to error tracking)
- NODE_ENV=production
- Debug features disabled

### Step 7: Audit Database Connection

Check database configuration:

```bash
# Connection string exposure
grep -rn "DATABASE_URL\|postgres://\|mysql://" --include="*.ts" --include="*.tsx"

# SSL configuration
grep -rn "ssl:\|sslmode" --include="*.ts"
```

**Required:**
- SSL/TLS for database connections
- Connection string in environment variables
- Connection pooling configured
- No plaintext credentials in code

### Step 8: Audit Third-Party Services

Check external service configuration:

```bash
# Webhook endpoints
grep -rn "webhook\|/api/hook" --include="*.ts"

# API keys and services
grep -rn "STRIPE\|RESEND\|SUPABASE\|OPENAI" --include="*.env*" --include="*.ts"
```

**For each service:**
- Webhook signature validation enabled
- API keys scoped to minimum permissions
- Production vs test keys properly separated

### Step 9: Audit CI/CD Pipeline

Check deployment pipeline:

```bash
# GitHub Actions
cat .github/workflows/*.yml 2>/dev/null

# Check for secrets in CI
grep -rn "secrets\.\|env:" .github/workflows/
```

**Required:**
- Secrets stored in CI/CD secrets manager
- No secrets in workflow files
- Branch protection on main/production
- Required reviews for deployments

### Step 10: Generate Report

```markdown
## Deployment Security Audit Report

### HTTPS & Transport
- [ ] All URLs use HTTPS: [PASS/FAIL]
- [ ] HTTP redirects to HTTPS: [PASS/FAIL]
- [ ] HSTS header configured: [PASS/FAIL]

### Security Headers
- [ ] X-Frame-Options: [PASS/FAIL]
- [ ] X-Content-Type-Options: [PASS/FAIL]
- [ ] Referrer-Policy: [PASS/FAIL]
- [ ] Strict-Transport-Security: [PASS/FAIL]

### Secrets Management
- [ ] No secrets in code: [PASS/FAIL]
- [ ] .env files gitignored: [PASS/FAIL]
- [ ] Production secrets in environment: [PASS/FAIL]
- [ ] No secrets in git history: [PASS/FAIL]

### Error Handling
- [ ] Custom 404 page: [PASS/FAIL]
- [ ] Custom 500 page: [PASS/FAIL]
- [ ] No stack traces exposed: [PASS/FAIL]

### Logging
- [ ] Console.logs removed: [PASS/FAIL]
- [ ] No secrets in logs: [PASS/FAIL]
- [ ] Structured logging: [PASS/FAIL]

### Build Configuration
- [ ] Source maps disabled: [PASS/FAIL]
- [ ] NODE_ENV=production: [PASS/FAIL]
- [ ] Debug features disabled: [PASS/FAIL]

### Infrastructure
- [ ] Database SSL enabled: [PASS/FAIL]
- [ ] Webhook signatures validated: [PASS/FAIL]
- [ ] CI/CD secrets properly managed: [PASS/FAIL]

### Critical Issues (must fix)
1. [Issue with file path and line number]
   - **Risk:** [What could go wrong]
   - **Fix:** [How to remediate]

### Warnings (should fix)
1. [Issue description]

### Recommendations
1. [Improvement suggestion]
```

## Platform-Specific Checks

### Vercel
```json
// vercel.json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "X-Content-Type-Options", "value": "nosniff" }
      ]
    }
  ]
}
```

- Environment variables in dashboard (not vercel.json)
- Preview deployments have different env vars
- Edge functions have size limits

### Netlify
```toml
# netlify.toml
[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-Content-Type-Options = "nosniff"
```

- Environment variables in dashboard
- Build plugins for security scanning

### Cloudflare Pages
- Security headers via _headers file or Functions
- Environment variables in dashboard
- WAF rules available on paid plans

## Common Vulnerabilities

### Missing HTTPS
**Risk:** Traffic intercepted, credentials stolen
**Fix:** Force HTTPS, configure HSTS

### Verbose Error Pages
**Risk:** Stack traces reveal system details
**Fix:** Custom error pages, log details server-side

### Secrets in Git
**Risk:** Anyone with repo access sees credentials
**Fix:** Use .gitignore, rotate leaked secrets immediately

### Missing Security Headers
**Risk:** Clickjacking, XSS, MIME attacks
**Fix:** Configure headers in hosting platform or middleware

### Source Maps in Production
**Risk:** Attackers can read original source code
**Fix:** Disable source maps or upload privately to error tracking

## Pre-Deployment Checklist

Before every production deployment:

- [ ] `npm audit` shows no critical issues
- [ ] All secrets in environment variables
- [ ] HTTPS enforced
- [ ] Security headers configured
- [ ] Custom error pages working
- [ ] Console.logs removed or replaced with logger
- [ ] Source maps disabled
- [ ] Database connection uses SSL
- [ ] Webhook endpoints validate signatures

## Tips

- Use securityheaders.com to scan your deployed site
- Check Vercel/Netlify deployment logs for warnings
- Set up Dependabot for automated dependency updates
- Use secret scanning in GitHub (Settings > Security)
- Test error pages by visiting /nonexistent-page

## How to Verify

### After Remediation
- Visit site with HTTP - should redirect to HTTPS
- Check response headers in DevTools > Network
- Visit /nonexistent - custom 404, no stack trace
- Trigger error - custom error page, no internals exposed
- Check git status - .env files not tracked

### External Scanning
1. Run securityheaders.com scan - grade A or better
2. Run SSL Labs test - grade A
3. Run `npm audit` - no critical vulnerabilities
4. Check GitHub security alerts - all resolved
