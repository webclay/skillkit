---
name: vercel-ai-sdk
description: Vercel AI SDK for streaming AI responses, chat interfaces, and tool calling. Trigger words - ai sdk, chat, chatbot, streaming, llm, gpt, claude, openai, anthropic, text generation, ai assistant, useChat, streamText, generateText, tool calling
---

# Vercel AI SDK

TypeScript toolkit for building AI-powered applications with streaming, tool calling, and multi-provider support.

## When to Use This Skill

- User asks to add AI features or chatbot
- package.json contains "ai" or "@ai-sdk/*"
- User mentions OpenAI, Anthropic, Claude, or GPT integration
- User wants streaming responses or chat UI
- User asks about tool calling or function calling

## Instructions

### Step 1: Install AI SDK

```bash
pnpm add ai @ai-sdk/openai
# or for Anthropic
pnpm add ai @ai-sdk/anthropic
```

### Step 2: Set Environment Variables

```env
OPENAI_API_KEY=sk-...
# or
ANTHROPIC_API_KEY=sk-ant-...
```

### Step 3: Create Chat API Route

**app/api/chat/route.ts:**
```ts
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = await streamText({
    model: openai('gpt-4-turbo'),
    system: 'You are a helpful assistant.',
    messages,
  });

  return result.toDataStreamResponse();
}
```

### Step 4: Create Chat Component

```tsx
'use client';
import { useChat } from 'ai/react';

export function Chat() {
  const { messages, input, handleInputChange, handleSubmit, isLoading } = useChat();

  return (
    <div>
      {messages.map(m => (
        <div key={m.id}>
          <strong>{m.role}:</strong> {m.content}
        </div>
      ))}
      <form onSubmit={handleSubmit}>
        <input value={input} onChange={handleInputChange} disabled={isLoading} />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

## Examples

**Basic Text Generation:**
```ts
import { generateText } from 'ai';
import { openai } from '@ai-sdk/openai';

const { text } = await generateText({
  model: openai('gpt-4-turbo'),
  prompt: 'Write a haiku about coding',
});
```

**Streaming:**
```ts
import { streamText } from 'ai';

const result = await streamText({
  model: openai('gpt-4-turbo'),
  prompt: 'Tell me a story',
});

for await (const chunk of result.textStream) {
  process.stdout.write(chunk);
}
```

**Structured Output:**
```ts
import { generateObject } from 'ai';
import { z } from 'zod';

const { object } = await generateObject({
  model: openai('gpt-4-turbo'),
  schema: z.object({
    name: z.string(),
    category: z.string(),
    price: z.number(),
  }),
  prompt: 'Extract product info from: "New headphones $199"',
});
```

**Tool Calling:**
```ts
import { generateText, tool } from 'ai';
import { z } from 'zod';

const result = await generateText({
  model: openai('gpt-4-turbo'),
  prompt: "What's the weather in Paris?",
  tools: {
    getWeather: tool({
      description: 'Get weather for a city',
      parameters: z.object({ city: z.string() }),
      execute: async ({ city }) => {
        return { temp: 22, condition: 'Sunny' };
      },
    }),
  },
});
```

**useChat Options:**
```tsx
const { messages, input, handleSubmit, reload, stop } = useChat({
  api: '/api/chat',
  onFinish: (message) => console.log('Done:', message),
  onError: (error) => console.error(error),
});
```

## Reference Files

- [streaming.md](streaming.md) - Streaming patterns
- [tools.md](tools.md) - Tool calling reference

## Provider Models

```ts
import { openai } from '@ai-sdk/openai';
import { anthropic } from '@ai-sdk/anthropic';
import { google } from '@ai-sdk/google';

openai('gpt-4-turbo')
anthropic('claude-3-5-sonnet-20241022')
google('gemini-1.5-pro')
```

## Tips

- Use `streamText` for chat, `generateText` for one-off completions
- Use `generateObject` for structured JSON output
- Lower temperature (0.1-0.3) for factual, higher (0.7-0.9) for creative
- Set `maxTokens` to control cost

## How to Verify

### Quick Checks
- `curl -X POST localhost:3000/api/chat -d '{"messages":[{"role":"user","content":"Hi"}]}'` - API responds
- Check browser Network tab shows streaming response
- No API key errors in terminal

### Manual Verification
- Type a message in chat UI - response streams in
- Messages display correctly with role labels
- Stop button works during streaming (if implemented)
- Reload button regenerates last response (if implemented)

### Common Issues
- "Invalid API key": Check env variable name matches provider (OPENAI_API_KEY, ANTHROPIC_API_KEY)
- No streaming: Ensure using `streamText` + `toDataStreamResponse()`
- useChat not updating: Verify API route returns correct streaming format
- Tool not executing: Check tool description is clear, parameters match schema
