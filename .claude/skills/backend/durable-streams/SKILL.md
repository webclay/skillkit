---
name: durable-streams
description: Use this skill when implementing resumable HTTP streaming with the Durable Streams protocol. Activate when the user mentions reliable streaming, stream recovery after disconnect, offset-based reconnection, or durable AI token streaming.
---

# Durable Streams

Open protocol for reliable, resumable HTTP streaming with offset-based recovery.

## When to Use This Skill

- AI chat/token streaming that survives disconnects
- Real-time data sync (works with Electric SQL)
- Event streaming that needs exactly-once delivery
- Mobile apps with unreliable networks
- Multi-device streaming (start on phone, continue on laptop)
- Collaborative real-time features

## When NOT to Use

- Simple request/response APIs
- Short-lived streams that don't need recovery
- WebSocket-only architectures

## Setup

```bash
# Client
pnpm add @durable-streams/client

# Server (if self-hosting)
pnpm add @durable-streams/server
```

Available clients: TypeScript, Python, Go, Elixir, C#, Swift, PHP, Java, Rust, Ruby

## Core Concept

Traditional streaming loses data on disconnect. Durable Streams assigns each chunk an **offset**, allowing clients to resume exactly where they left off.

```
Client: GET /stream?offset=0&live=true
Server: [chunk 0] [chunk 1] [chunk 2]...
        Header: Stream-Next-Offset: 42

[network disconnect]

Client: GET /stream?offset=42&live=true  # Resume from offset 42
Server: [chunk 42] [chunk 43]...         # No duplicates, no gaps
```

## Protocol Basics

### Endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| PUT | `/v1/stream/{name}` | Create stream |
| POST | `/v1/stream/{name}` | Append data |
| GET | `/v1/stream/{name}?offset=X&live=MODE` | Read stream |

### Key Headers

- `Stream-Next-Offset` - Resume point for exactly-once delivery
- `Cache-Control` - Enables CDN caching for historical reads
- `Content-Type` - Preserved across all reads

### Query Parameters

- `offset` - Start position (-1 for beginning)
- `live` - Mode: `false` (catch-up only), `true` (wait for new), `auto` (both)

## Client Usage

### Basic Read

```typescript
import { StreamHandle } from "@durable-streams/client"

const handle = new StreamHandle("https://api.example.com/v1/stream/my-stream")

// Read from beginning
const res = await handle.stream({ offset: -1, live: false })
const data = await res.text()
```

### Live Streaming with Auto-Resume

```typescript
const res = await handle.stream({ live: "auto" })

res.subscribeJson(async (batch) => {
  for (const item of batch.items) {
    console.log("Received:", item)
  }
})

// Save offset for later resume
const savedOffset = res.offset
```

### Resume from Saved Offset

```typescript
// Later, or on different device
const resumed = await handle.stream({
  offset: savedOffset,
  live: "auto"
})
```

### Idempotent Producer (Fire-and-Forget Writes)

```typescript
import { IdempotentProducer } from "@durable-streams/client"

const producer = new IdempotentProducer(stream, "service-1", {
  autoClaim: true,
  onError: (err) => console.error("Batch failed:", err)
})

for (const event of events) {
  producer.append(event)  // Auto-batched, guaranteed delivery
}
await producer.flush()
```

## Server Implementation

### Using @durable-streams/server

```typescript
import { createServer } from "@durable-streams/server"

const server = createServer({
  storage: "memory",  // or "postgres", "redis"
})

server.listen(3000)
```

### Custom Implementation (API Route)

```typescript
// app/api/stream/[name]/route.ts
import { NextRequest } from "next/server"

const streams = new Map<string, { chunks: string[], listeners: Set<any> }>()

export async function GET(req: NextRequest, { params }: { params: { name: string } }) {
  const { searchParams } = new URL(req.url)
  const offset = parseInt(searchParams.get("offset") || "-1")
  const live = searchParams.get("live") === "true"

  const stream = streams.get(params.name)
  if (!stream) return new Response("Not found", { status: 404 })

  const startFrom = offset === -1 ? 0 : offset
  const encoder = new TextEncoder()

  const readable = new ReadableStream({
    start(controller) {
      // Send historical chunks
      for (let i = startFrom; i < stream.chunks.length; i++) {
        controller.enqueue(encoder.encode(`id: ${i}\ndata: ${stream.chunks[i]}\n\n`))
      }

      if (!live) {
        controller.close()
        return
      }

      // Listen for new chunks
      const listener = (chunk: string, id: number) => {
        if (id >= startFrom) {
          controller.enqueue(encoder.encode(`id: ${id}\ndata: ${chunk}\n\n`))
        }
      }
      stream.listeners.add(listener)
    }
  })

  return new Response(readable, {
    headers: {
      "Content-Type": "text/event-stream",
      "Cache-Control": "no-cache",
      "Stream-Next-Offset": String(stream.chunks.length)
    }
  })
}

export async function POST(req: NextRequest, { params }: { params: { name: string } }) {
  const data = await req.text()

  let stream = streams.get(params.name)
  if (!stream) {
    stream = { chunks: [], listeners: new Set() }
    streams.set(params.name, stream)
  }

  const id = stream.chunks.length
  stream.chunks.push(data)
  stream.listeners.forEach(l => l(data, id))

  return new Response(null, {
    status: 204,
    headers: { "Stream-Next-Offset": String(id + 1) }
  })
}
```

## AI Token Streaming Example

```typescript
// Server: Stream AI response with durability
export async function POST(req: Request) {
  const { messages, streamId, resumeOffset } = await req.json()

  const stream = await anthropic.messages.stream({
    model: "claude-sonnet-4-20250514",
    max_tokens: 1024,
    messages,
  })

  let offset = 0
  const encoder = new TextEncoder()

  const readable = new ReadableStream({
    async start(controller) {
      for await (const event of stream) {
        if (offset >= (resumeOffset || 0)) {
          const data = JSON.stringify({ offset, ...event })
          controller.enqueue(encoder.encode(`id: ${offset}\ndata: ${data}\n\n`))
        }
        offset++
      }
      controller.close()
    }
  })

  return new Response(readable, {
    headers: {
      "Content-Type": "text/event-stream",
      "Stream-Next-Offset": String(offset)
    }
  })
}
```

```typescript
// Client: React hook with auto-resume
function useDurableAIStream() {
  const [content, setContent] = useState("")
  const [offset, setOffset] = useState(0)

  const stream = useCallback(async (messages: any[], resumeFrom?: number) => {
    const res = await fetch("/api/chat", {
      method: "POST",
      body: JSON.stringify({
        messages,
        resumeOffset: resumeFrom
      })
    })

    const reader = res.body!.getReader()
    const decoder = new TextDecoder()

    while (true) {
      const { done, value } = await reader.read()
      if (done) break

      const text = decoder.decode(value)
      const matches = text.matchAll(/id: (\d+)\ndata: (.+)\n/g)

      for (const match of matches) {
        const eventOffset = parseInt(match[1])
        const event = JSON.parse(match[2])

        setOffset(eventOffset + 1)

        if (event.type === "content_block_delta") {
          setContent(prev => prev + event.delta.text)
        }
      }
    }
  }, [])

  const resume = useCallback((messages: any[]) => {
    return stream(messages, offset)
  }, [stream, offset])

  return { content, offset, stream, resume }
}
```

## Integration with Electric SQL

Electric SQL uses Durable Streams internally for Postgres sync. The same protocol powers both:

- Electric: Database change streaming
- Your app: AI tokens, events, real-time updates

```typescript
// Electric uses the protocol for shape streaming
const shape = new ShapeStream({
  url: `${ELECTRIC_URL}/v1/shape`,
  params: { table: 'items' },
  // Automatically handles offset-based resumption
})
```

## Tips

- Store offset in localStorage for page refresh recovery
- Use `live: "auto"` for catch-up + real-time in one request
- CDN-friendly: historical reads can be cached
- Works with any content type (JSON, binary, text)
- Caddy plugin available for production deployments

## How to Verify

### Quick Checks
- Stream starts and data appears incrementally
- Disconnect and resume continues from saved offset
- No duplicate or missing data after resume

### Common Issues
- Offset not saved: Store `Stream-Next-Offset` header value
- Duplicates on resume: Ensure server skips chunks before offset
- Connection drops: Add reconnection with exponential backoff
