---
name: durable-streams
description: Durable Streams for resumable AI token streaming. Trigger words - durable streams, resumable streaming, reconnect stream, offline streaming, reliable streaming, stream recovery
---

# Durable Streams

Resumable streaming protocol for AI applications that survives network interruptions.

## When to Use This Skill

- Building AI chat with unreliable networks (mobile)
- Long-running AI generations that might timeout
- Need to resume streams after page refresh
- Want guaranteed delivery of AI tokens

## When NOT to Use

- Simple chatbots on stable connections
- Non-streaming AI responses
- Server-to-server AI calls

## Setup

```bash
pnpm add @anthropic-ai/sdk
```

## Core Concept

Traditional streaming breaks on disconnect. Durable Streams assign each chunk an ID, allowing clients to resume from where they left off.

```
Client: "Give me chunks starting from #42"
Server: "Here's #42, #43, #44..."
[disconnect]
Client: "Give me chunks starting from #45"
Server: "Here's #45, #46..."
```

## Server Implementation

```typescript
// app/api/chat/route.ts
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic();

export async function POST(req: Request) {
  const { messages, lastEventId } = await req.json();

  const stream = await client.messages.stream({
    model: "claude-sonnet-4-20250514",
    max_tokens: 1024,
    messages,
  });

  // If resuming, skip already-sent events
  let eventId = 0;
  const startFrom = lastEventId ? parseInt(lastEventId) + 1 : 0;

  const encoder = new TextEncoder();
  const readable = new ReadableStream({
    async start(controller) {
      for await (const event of stream) {
        if (eventId >= startFrom) {
          const data = JSON.stringify({ id: eventId, ...event });
          controller.enqueue(encoder.encode(`id: ${eventId}\ndata: ${data}\n\n`));
        }
        eventId++;
      }
      controller.close();
    },
  });

  return new Response(readable, {
    headers: {
      "Content-Type": "text/event-stream",
      "Cache-Control": "no-cache",
      "Connection": "keep-alive",
    },
  });
}
```

## Client Implementation

```typescript
// hooks/useDurableStream.ts
import { useState, useCallback, useRef } from "react";

export function useDurableStream() {
  const [content, setContent] = useState("");
  const [isStreaming, setIsStreaming] = useState(false);
  const lastEventIdRef = useRef<string | null>(null);
  const abortRef = useRef<AbortController | null>(null);

  const startStream = useCallback(async (messages: any[], resume = false) => {
    setIsStreaming(true);
    if (!resume) {
      setContent("");
      lastEventIdRef.current = null;
    }

    abortRef.current = new AbortController();

    const response = await fetch("/api/chat", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        messages,
        lastEventId: resume ? lastEventIdRef.current : null,
      }),
      signal: abortRef.current.signal,
    });

    const reader = response.body!.getReader();
    const decoder = new TextDecoder();
    let buffer = "";

    while (true) {
      const { done, value } = await reader.read();
      if (done) break;

      buffer += decoder.decode(value, { stream: true });
      const lines = buffer.split("\n\n");
      buffer = lines.pop() || "";

      for (const line of lines) {
        const idMatch = line.match(/^id: (\d+)/);
        const dataMatch = line.match(/data: (.+)$/m);

        if (idMatch) lastEventIdRef.current = idMatch[1];
        if (dataMatch) {
          const event = JSON.parse(dataMatch[1]);
          if (event.type === "content_block_delta") {
            setContent((prev) => prev + event.delta.text);
          }
        }
      }
    }

    setIsStreaming(false);
  }, []);

  const resumeStream = useCallback((messages: any[]) => {
    return startStream(messages, true);
  }, [startStream]);

  return { content, isStreaming, startStream, resumeStream };
}
```

## Usage

```tsx
function Chat() {
  const { content, isStreaming, startStream, resumeStream } = useDurableStream();
  const [messages, setMessages] = useState([]);

  const handleSend = async (text: string) => {
    const newMessages = [...messages, { role: "user", content: text }];
    setMessages(newMessages);
    await startStream(newMessages);
  };

  const handleResume = () => resumeStream(messages);

  return (
    <div>
      <div>{content}</div>
      {!isStreaming && content && (
        <button onClick={handleResume}>Resume</button>
      )}
    </div>
  );
}
```

## Tips

- Store `lastEventId` in localStorage for page refresh recovery
- Add exponential backoff for auto-reconnection
- Consider server-side event storage for longer durability
- Works with any streaming AI provider, not just Anthropic

## How to Verify

### Quick Checks
- Stream starts and tokens appear incrementally
- Disconnecting and resuming continues from last position
- No duplicate tokens after resume

### Common Issues
- Events not resuming: Check `lastEventId` is being sent correctly
- Duplicate content: Ensure server skips events before `startFrom`
- Connection drops: Add reconnection logic with backoff
