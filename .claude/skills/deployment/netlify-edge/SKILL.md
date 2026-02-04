---
name: netlify-edge
description: Netlify deployment with Edge Functions and Supabase integration. Trigger words - netlify, netlify edge, edge function, deploy netlify, netlify functions
---

# Netlify + Edge Functions

Netlify deployment with edge functions for secure API proxying and global performance.

## When to Use This Skill

- Deploying Next.js to Netlify
- Need to proxy external API calls securely
- Want edge functions for low-latency processing
- Using Supabase Edge Functions for backend

## When NOT to Use

- Next.js-optimized deployment (use Vercel)
- Need complex serverless orchestration

## netlify.toml Configuration

```toml
[build]
  command = "pnpm build"
  publish = ".next"
  base = "/"

[context.production]
  branch = "main"

[build.environment]
  NODE_VERSION = "20"

# Security headers
[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-Content-Type-Options = "nosniff"
    Referrer-Policy = "strict-origin-when-cross-origin"

# Cache static assets
[[headers]]
  for = "/_next/static/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"
```

## Next.js Image Optimization

Netlify doesn't support Next.js image optimization by default:

```tsx
// Add unoptimized prop to all Image components
import Image from 'next/image';

<Image
  src="/images/logo.png"
  alt="Logo"
  width={200}
  height={50}
  unoptimized  // Required for Netlify
/>
```

## Supabase Edge Functions Architecture

Keep API keys secure by proxying through Supabase:

```
Next.js → Supabase Edge Function → External API (OpenRouter, etc.)
```

### Edge Function Pattern

```typescript
// supabase/functions/openrouter-api/index.ts
import { serve } from "https://deno.land/std@0.203.0/http/server.ts";
import { createClient } from "https://esm.sh/@supabase/supabase-js@2.39.7";

serve(async (req) => {
  try {
    const { user_id, model, system_prompt, user_prompt } = await req.json();

    // Check credits before API call
    const supabase = createClient(
      Deno.env.get("SUPABASE_URL"),
      Deno.env.get("SUPABASE_SERVICE_ROLE_KEY")
    );

    const { data: profile } = await supabase
      .from('profiles')
      .select('credits_balance')
      .eq('id', user_id)
      .single();

    if ((profile?.credits_balance || 0) < 1) {
      return new Response(JSON.stringify({
        error: 'Insufficient credits'
      }), { status: 402 });
    }

    // Call external API
    const response = await fetch("https://openrouter.ai/api/v1/chat/completions", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${Deno.env.get("OPENROUTER_API_KEY")}`,
      },
      body: JSON.stringify({
        model,
        messages: [
          { role: "system", content: system_prompt },
          { role: "user", content: user_prompt }
        ]
      })
    });

    const data = await response.json();

    // Deduct credits after success
    await supabase.rpc('deduct_credits', {
      p_user_id: user_id,
      p_amount: 1,
      p_type: 'ai_generation'
    });

    return new Response(JSON.stringify({
      result: data.choices[0]?.message?.content,
      usage: data.usage
    }), { status: 200 });

  } catch (error) {
    return new Response(JSON.stringify({
      error: error.message
    }), { status: 500 });
  }
});
```

### Calling from Next.js

```typescript
// app/api/generate/route.ts
export async function POST(request: Request) {
  const body = await request.json();
  const session = await getSession();

  if (!session) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const response = await fetch(
    `${process.env.NEXT_PUBLIC_SUPABASE_URL}/functions/v1/openrouter-api`,
    {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY}`
      },
      body: JSON.stringify({
        user_id: session.user.id,
        model: body.model,
        system_prompt: body.system_prompt,
        user_prompt: body.user_prompt
      })
    }
  );

  return Response.json(await response.json());
}
```

## Environment Variables

**Netlify UI:**
```bash
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbG...
```

**Supabase Edge Functions (in Dashboard):**
```bash
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_SERVICE_ROLE_KEY=eyJhbG...
OPENROUTER_API_KEY=sk-or-v1-...
```

## Tips

- All API keys in Supabase, not Next.js
- Use `unoptimized` on all Image components
- Deploy from `main` or `dev` branch
- Set environment variables in Netlify UI
- Edge functions handle credit checking and deduction

## How to Verify

### Quick Checks
- Build passes with `pnpm build`
- Deploy completes in Netlify dashboard
- Images load without 400 errors
- Edge function calls succeed

### Common Issues
- Image 400 errors: Add `unoptimized` prop
- API key not found: Check Supabase secrets
- CORS errors: Add proper headers in Edge Function
