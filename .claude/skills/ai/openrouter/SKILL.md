---
name: openrouter
description: OpenRouter multi-model AI API. Trigger words - openrouter, multi-model, model fallback, llm api, ai gateway, model routing, cost optimization ai
---

# OpenRouter

Unified API for 100+ LLM models with automatic fallbacks, load balancing, and cost optimization.

## When to Use This Skill

- User wants to access multiple AI models through one API
- User mentions OpenRouter, model routing, or LLM fallbacks
- User needs cost-optimized AI API calls
- User wants to compare outputs from different models

## Setup

### Install

```bash
pnpm add openai  # OpenRouter is OpenAI-compatible
```

### Environment Variables

```env
OPENROUTER_API_KEY=sk-or-v1-xxxxx
OPENROUTER_SITE_URL=https://yourapp.com  # Optional
OPENROUTER_SITE_NAME=YourApp             # Optional
```

### Create Client

**lib/openrouter.ts:**
```typescript
import OpenAI from 'openai';

export const openrouter = new OpenAI({
  baseURL: 'https://openrouter.ai/api/v1',
  apiKey: process.env.OPENROUTER_API_KEY,
  defaultHeaders: {
    'HTTP-Referer': process.env.OPENROUTER_SITE_URL,
    'X-Title': process.env.OPENROUTER_SITE_NAME,
  }
});
```

## Examples

**Basic Chat:**
```typescript
const completion = await openrouter.chat.completions.create({
  model: 'anthropic/claude-3.5-sonnet',
  messages: [
    { role: 'system', content: 'You are a helpful assistant.' },
    { role: 'user', content: userMessage }
  ]
});

return completion.choices[0].message.content;
```

**Streaming:**
```typescript
const stream = await openrouter.chat.completions.create({
  model: 'anthropic/claude-3.5-sonnet',
  messages,
  stream: true
});

for await (const chunk of stream) {
  const content = chunk.choices[0]?.delta?.content || '';
  process.stdout.write(content);
}
```

**Model Fallbacks:**
```typescript
const completion = await openrouter.chat.completions.create({
  model: 'anthropic/claude-3.5-sonnet',
  messages,
  route: 'fallback',
  models: [
    'anthropic/claude-3.5-sonnet',
    'openai/gpt-4-turbo',
    'google/gemini-pro-1.5'
  ]
});
```

**Auto Cost Optimization:**
```typescript
const completion = await openrouter.chat.completions.create({
  model: 'openrouter/auto',  // Let OpenRouter choose
  messages,
  route: 'cheapest'  // or 'fastest', 'balanced'
});
```

## Popular Models

```typescript
// Anthropic
'anthropic/claude-3.5-sonnet'  // Best overall
'anthropic/claude-3-haiku'     // Fast, cheap

// OpenAI
'openai/gpt-4-turbo'
'openai/gpt-3.5-turbo'

// Google
'google/gemini-pro-1.5'        // 2M token context

// Meta
'meta-llama/llama-3.1-70b-instruct'
```

## Tips

- Use `openrouter/auto` for cost optimization
- Vision models support image inputs (Claude 3, GPT-4V, Gemini)
- Set `response_format: { type: 'json_object' }` for structured output
- Never expose API key to client - use server-side routes

## How to Verify

### Quick Checks
- API call returns completion without error
- Check OpenRouter Dashboard shows request logged
- No API key errors in terminal

### Manual Verification
- Test with different models to confirm routing works
- Verify streaming responses arrive incrementally
- Check cost tracking in dashboard

### Common Issues
- "Invalid API key": Check OPENROUTER_API_KEY is correct
- "Model not found": Verify model ID format (provider/model-name)
- Rate limited: Implement exponential backoff
