---
name: vercel
description: Vercel deployment platform for Next.js and web applications. Trigger words - vercel, deploy, deployment, preview deployment, serverless function, edge runtime, vercel.json
---

# Vercel

Zero-config deployment platform with global CDN, serverless functions, and edge computing.

## When to Use This Skill

- Deploying Next.js applications
- Need preview deployments for every PR
- Want zero-config CI/CD from Git
- Building global apps requiring edge computing
- Need automatic scaling without DevOps

## When NOT to Use

- Need persistent servers or long-running processes
- Require custom Docker containers
- Building non-web applications

## Deployment Methods

### Git Integration (Recommended)

1. Visit [vercel.com/new](https://vercel.com/new)
2. Import repository (GitHub, GitLab, Bitbucket)
3. Configure settings
4. Deploy

**Automatic deploys:**
- Push to `main` → production deployment
- Push to other branches → preview deployment with unique URL
- PRs get automatic preview with status checks

### Vercel CLI

```bash
npm i -g vercel

# Deploy to preview
vercel

# Deploy to production
vercel --prod

# Pull environment variables
vercel env pull .env.local
```

## Environment Variables

```bash
# Via CLI
vercel env add DATABASE_URL
vercel env pull .env.local

# Types: Production, Preview, Development
```

| Variable | Production | Preview |
|----------|-----------|---------|
| DATABASE_URL | prod-db.com | staging-db.com |
| STRIPE_SECRET_KEY | sk_live_... | sk_test_... |

**System variables available:**
```typescript
process.env.VERCEL          // "1" if on Vercel
process.env.VERCEL_ENV      // "production" | "preview"
process.env.VERCEL_URL      // Deployment URL
process.env.VERCEL_REGION   // Region code
```

## Serverless Functions

```typescript
// app/api/hello/route.ts
export const runtime = 'nodejs'
export const maxDuration = 60  // Pro: up to 900s

export async function GET(request: Request) {
  return Response.json({ message: 'Hello!' })
}
```

## Edge Functions

```typescript
// app/api/edge/route.ts
export const runtime = 'edge'

export async function GET(request: Request) {
  const country = request.headers.get('x-vercel-ip-country')
  return Response.json({ country })
}
```

## vercel.json Configuration

```json
{
  "buildCommand": "pnpm build",
  "installCommand": "pnpm install",
  "framework": "nextjs",
  "functions": {
    "api/**/*.ts": {
      "memory": 1024,
      "maxDuration": 30
    }
  },
  "crons": [
    {
      "path": "/api/cron/cleanup",
      "schedule": "0 0 * * *"
    }
  ],
  "redirects": [
    {
      "source": "/old-path",
      "destination": "/new-path",
      "permanent": true
    }
  ],
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        { "key": "Access-Control-Allow-Origin", "value": "*" }
      ]
    }
  ]
}
```

## Cron Jobs

```typescript
// app/api/cron/cleanup/route.ts
export async function GET(request: Request) {
  // Verify cron secret
  const auth = request.headers.get('authorization')
  if (auth !== `Bearer ${process.env.CRON_SECRET}`) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 })
  }

  await cleanupOldData()
  return Response.json({ success: true })
}
```

## Custom Domains

1. Project Settings → Domains
2. Add domain: `example.com`
3. Configure DNS:
   - A Record: `76.76.21.21`
   - Or CNAME: `cname.vercel-dns.com`

## Tips

- Use `vercel --prod` for production deploys
- Edge runtime for auth/routing (ultra-low latency)
- Serverless for database queries and heavy computation
- Set `maxDuration` for long-running tasks
- Environment changes require redeploy

## How to Verify

### Quick Checks
- `vercel` deploys successfully
- Preview URL accessible for branches
- Functions respond at `/api/*` endpoints

### Common Issues
- "Function timeout": Increase `maxDuration` or optimize code
- "Environment variable not found": Redeploy after adding
- "404 on API route": Check file location and exports
