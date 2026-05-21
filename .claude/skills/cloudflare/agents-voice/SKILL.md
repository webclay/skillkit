---
name: cloudflare-agents-voice
description: Build real-time voice AI agents on Cloudflare Workers. Covers withVoice and withVoiceInput mixins, STT providers (WorkersAIFluxSTT, WorkersAINova3STT), TTS, barge-in interruption, onTurn handler, React hooks (useVoiceAgent, useVoiceInput), and the VoiceClient. Trigger words - cloudflare voice, voice agent, cloudflare STT, cloudflare TTS, real-time audio, withVoice, withVoiceInput, WorkersAIFluxSTT, WorkersAINova3STT, WorkersAITTS, useVoiceAgent, useVoiceInput, VoiceClient, barge-in, voice pipeline, speech to text cloudflare, text to speech cloudflare, realtime voice agent, audio agent
---

# Cloudflare Voice Agents

Build real-time voice AI agents on Cloudflare Workers using `@cloudflare/voice`. The package adds continuous STT, LLM turn handling, streaming TTS, barge-in interruption, and conversation persistence on top of the Agents SDK.

> **Experimental** - API is under active development and will break between releases. Pin your version and expect to rewrite when upgrading.

GitHub: https://github.com/cloudflare/agents/tree/main/packages/voice
Official docs: https://developers.cloudflare.com/agents/

## Prerequisites

Read [cloudflare/agents](../agents/SKILL.md) for the base `Agent` class, wrangler.jsonc setup, routing, and deployment fundamentals.

## Install

```bash
npm install @cloudflare/voice agents
```

**Pin the version** - `@cloudflare/voice` has breaking changes between releases:

```json
{
  "dependencies": {
    "@cloudflare/voice": "0.2.0"
  }
}
```

## Exports

| Export path | What it provides |
|---|---|
| `@cloudflare/voice` | Server-side mixins (`withVoice`, `withVoiceInput`), provider types, Workers AI providers |
| `@cloudflare/voice/react` | React hooks (`useVoiceAgent`, `useVoiceInput`) |
| `@cloudflare/voice/client` | Framework-agnostic `VoiceClient` class |

## Full Voice Pipeline: withVoice

`withVoice(Agent)` adds the complete pipeline: continuous STT, LLM turn handling, streaming TTS, barge-in interruption, and conversation persistence.

When the transcriber detects speech, any active LLM/TTS work is aborted and the client is told to stop queued audio playback - so users can interrupt the agent before a final transcript arrives.

```typescript
import { Agent } from "agents";
import {
  withVoice,
  WorkersAIFluxSTT,
  WorkersAITTS,
  type VoiceTurnContext,
} from "@cloudflare/voice";
import { routeAgentRequest } from "agents";

const VoiceAgent = withVoice(Agent);

export class MyVoiceAgent extends VoiceAgent<Env> {
  transcriber = new WorkersAIFluxSTT(this.env.AI);
  tts = new WorkersAITTS(this.env.AI);

  async onTurn(transcript: string, context: VoiceTurnContext) {
    return "Hello! I heard you say: " + transcript;
  }
}

export default {
  async fetch(request: Request, env: Env, ctx: ExecutionContext) {
    return (await routeAgentRequest(request, env)) ?? new Response("Not found", { status: 404 });
  },
} satisfies ExportedHandler<Env>;
```

## Streaming onTurn Responses

`onTurn()` can return streaming text directly from the AI SDK - TTS consumes the stream as it arrives:

```typescript
import { streamText } from "ai";
import { createWorkersAI } from "workers-ai-provider";

async onTurn(transcript: string, context: VoiceTurnContext) {
  const workersai = createWorkersAI({ binding: this.env.AI });

  const { textStream } = streamText({
    model: workersai("@cf/meta/llama-4-scout-17b-16e-instruct"),
    system: "You are a helpful voice assistant. Be concise - this is audio.",
    messages: [
      ...context.history,
      { role: "user", content: transcript },
    ],
  });

  return textStream;
}
```

Use short, conversational responses - TTS will read everything out loud.

## STT Providers

### WorkersAIFluxSTT (recommended for withVoice)

Continuous per-call STT sessions with turn detection by the model. Recommended for full voice agents with barge-in:

```typescript
transcriber = new WorkersAIFluxSTT(this.env.AI);
```

Flux preserves the latest non-empty transcript and drives server-side barge-in via `StartOfTurn` events.

### WorkersAINova3STT (recommended for withVoiceInput)

Continuous STT with streaming segment normalization:

```typescript
transcriber = new WorkersAINova3STT(this.env.AI);
```

## TTS Providers

```typescript
import { WorkersAITTS } from "@cloudflare/voice";

tts = new WorkersAITTS(this.env.AI);
```

## wrangler.jsonc

Requires the Workers AI binding. Voice agents also need `keepAlive` to prevent DO eviction during active calls:

```jsonc
{
  "name": "voice-agent",
  "main": "src/server.ts",
  "compatibility_flags": ["nodejs_compat"],
  "ai": {
    "binding": "AI"
  },
  "durable_objects": {
    "bindings": [
      { "name": "MyVoiceAgent", "class_name": "MyVoiceAgent" }
    ]
  },
  "migrations": [
    { "tag": "v1", "new_sqlite_classes": ["MyVoiceAgent"] }
  ]
}
```

The voice mixin calls `keepAlive()` automatically during active calls to prevent Durable Object eviction.

## Input-Only Pipeline: withVoiceInput

`withVoiceInput(Agent)` adds only STT without TTS or full LLM turn handling. Useful for transcription, dictation, or voice-controlled agents where you handle audio output yourself:

```typescript
import { withVoiceInput, WorkersAINova3STT } from "@cloudflare/voice";

const InputAgent = withVoiceInput(Agent);

export class DictationAgent extends InputAgent<Env> {
  transcriber = new WorkersAINova3STT(this.env.AI);

  async onTranscript(transcript: string) {
    // Receive transcripts, handle output yourself
    this.broadcast(JSON.stringify({ type: "transcript", text: transcript }));
  }
}
```

## React Integration

### useVoiceAgent

Full voice pipeline hook - connects, manages audio input, and handles TTS playback:

```tsx
import { useVoiceAgent } from "@cloudflare/voice/react";

function VoiceUI() {
  const { status, start, stop } = useVoiceAgent({
    agent: "MyVoiceAgent",
    name: "session-1",
    enabled: true, // Set false to delay connecting until ready (e.g., waiting for auth)
  });

  return (
    <div>
      <p>Status: {status}</p>
      {status === "idle" ? (
        <button onClick={start}>Start Voice</button>
      ) : (
        <button onClick={stop}>Stop</button>
      )}
    </div>
  );
}
```

The `enabled` option delays creating and connecting the `VoiceClient` until prerequisites (like capability tokens) are ready.

### useVoiceInput

Input-only hook for transcription:

```tsx
import { useVoiceInput } from "@cloudflare/voice/react";

function TranscriptUI() {
  const { transcript, start, stop } = useVoiceInput({
    agent: "DictationAgent",
    name: "session-1",
  });

  return (
    <div>
      <button onClick={start}>Start Dictating</button>
      <button onClick={stop}>Stop</button>
      <p>{transcript}</p>
    </div>
  );
}
```

## Vanilla JavaScript: VoiceClient

Framework-agnostic client for non-React environments:

```typescript
import { VoiceClient } from "@cloudflare/voice/client";

const client = new VoiceClient({
  agent: "MyVoiceAgent",
  name: "session-1",
  onTranscript: (text) => console.log("Heard:", text),
  onAudio: (audioChunk) => playAudio(audioChunk),
  onStatusChange: (status) => updateUI(status),
});

await client.connect();
await client.startCall();
```

## Barge-In and Interruption

Barge-in is handled automatically by `withVoice`. When `StartOfTurn` fires (user starts speaking):

1. Active LLM streaming is aborted
2. TTS playback is cancelled on the client
3. The pipeline waits for the complete user transcript
4. `onTurn()` is called with the new transcript

No configuration needed - this is the default behavior with `WorkersAIFluxSTT`.

## Runtime Model Switching

Override `createTranscriber(connection)` to select STT model per connection (e.g. based on query params):

```typescript
createTranscriber(connection: Connection) {
  const url = new URL(connection.request?.url ?? "");
  const model = url.searchParams.get("model") ?? "flux";

  if (model === "nova") {
    return new WorkersAINova3STT(this.env.AI);
  }
  return new WorkersAIFluxSTT(this.env.AI);
}
```

Pass query params from the client:

```typescript
const { start } = useVoiceAgent({
  agent: "MyVoiceAgent",
  query: { model: "nova" },
});
```

## SFU Utilities

`@cloudflare/voice` also exports SFU (Selective Forwarding Unit) utilities for advanced multi-party audio scenarios. These are low-level building blocks for routing audio between multiple WebSocket clients. Consult the GitHub source for the current API as this surface is evolving.

## Common Gotchas

1. **Pin the version** - `@cloudflare/voice` has breaking changes between minor releases. Set an exact version in package.json.

2. **Use concise responses** - TTS reads everything aloud. Aim for 1-3 sentences per response.

3. **Flux requires per-call sessions** - `WorkersAIFluxSTT` creates a new STT session at `start_call` and keeps it alive for the call duration. The model handles turn detection - no client-side VAD needed.

4. **Duplicate `start_call` is silently ignored** - calling `start_call` while already in a call does nothing. Safe to retry.

5. **keepAlive is automatic** - the voice mixin calls `keepAlive()` during active calls so the DO doesn't hibernate mid-conversation. Don't add redundant keepAlive calls.

6. **AI binding is required** - even if using an external model for LLM, Workers AI binding is needed for the STT/TTS providers unless you bring your own.
