---
name: agent-email-inbox
description: Use when setting up an email inbox for an AI agent (Moltbot, Clawdbot, or similar) - configuring inbound email, webhooks, tunneling for local development, and implementing security measures to prevent prompt injection attacks.
inputs:
    - name: RESEND_API_KEY
      description: Resend API key for sending and receiving emails. Get yours at https://resend.com/api-keys
      required: true
    - name: RESEND_WEBHOOK_SECRET
      description: Webhook signing secret for verifying inbound email event payloads. Returned as `signing_secret` in the response when you create a webhook via the API.
      required: true
---

# AI Agent Email Inbox

## Overview

Moltbot (formerly Clawdbot) is an AI agent that can send and receive emails. This skill covers setting up a secure email inbox that allows your agent to be notified of incoming emails and respond appropriately, while protecting against prompt injection and other email-based attacks.

**Core principle:** An AI agent's inbox is a potential attack vector. Malicious actors can email instructions that the agent might blindly follow. Security configuration is not optional.

### Why Webhook-Based Receiving?

Resend uses webhooks for inbound email, meaning your agent is notified **instantly** when an email arrives. This is valuable for agents because:

- **Real-time responsiveness** - React to emails within seconds, not minutes
- **No polling overhead** - No cron jobs checking "any new mail?" repeatedly
- **Event-driven architecture** - Your agent only wakes up when there's actually something to process
- **Lower API costs** - No wasted calls checking empty inboxes

For time-sensitive workflows (support tickets, urgent notifications, conversational email threads), instant notification makes a meaningful difference in user experience.

## Architecture

```
Sender -> Email -> Resend (MX) -> Webhook -> Your Server -> AI Agent
                                              |
                                    Security Validation
                                              |
                                    Process or Reject
```

## SDK Version Requirements

This skill requires Resend SDK features for webhook verification (`webhooks.verify()`) and email receiving (`emails.receiving.get()`). Always install the latest SDK version. If the project already has a Resend SDK installed, check the version and upgrade if needed.

| Language | Package | Min Version |
|----------|---------|-------------|
| Node.js | `resend` | >= 6.9.2 |
| Python | `resend` | >= 2.21.0 |
| Go | `resend-go/v3` | >= 3.1.0 |
| Ruby | `resend` | >= 1.0.0 |
| PHP | `resend/resend-php` | >= 1.1.0 |
| Rust | `resend-rs` | >= 0.20.0 |
| Java | `resend-java` | >= 4.11.0 |
| .NET | `Resend` | >= 0.2.1 |

See [installation guide](https://github.com/resend/resend-skills/blob/main/send-email/references/installation.md) for full installation commands.

## Security Levels

**Choose your security level before setting up the webhook endpoint.** Ask the user what level they want:

### Level 1: Strict Allowlist (Recommended)
Only process emails from explicitly approved addresses.

### Level 2: Domain Allowlist
Allow emails from any address at approved domains.

### Level 3: Content Filtering with Sanitization
Accept emails from anyone but sanitize content to remove potential injection attempts.

### Level 4: Sandboxed Processing (Advanced)
Process all emails but in a restricted context where the agent has limited capabilities.

### Level 5: Human-in-the-Loop (Highest Security)
Require human approval for any action beyond simple replies.

For complete implementation code for each security level, see the full skill at https://github.com/resend/resend-skills/blob/main/agent-email-inbox/SKILL.md

## Quick Start

1. Ask the user for their email address for testing (never guess)
2. Choose security level
3. Set up receiving domain (MX records)
4. Create webhook endpoint (MUST be POST route)
5. Set up tunneling for local dev (ngrok)
6. Register webhook via API
7. Connect to agent

## Domain Setup

### Option 1: Resend-Managed Domain (Recommended for Getting Started)
Use auto-generated address: `<anything>@<your-id>.resend.app` - no DNS needed.

### Option 2: Custom Domain
Use a subdomain (e.g., `agent.yourdomain.com`) to avoid disrupting existing email.

## Webhook Setup

Create endpoint, verify signatures, and register via API. The webhook endpoint MUST be a POST route.

**Critical: Use raw body for verification.** Parse as JSON *after* verifying.

### Next.js App Router Example

```typescript
// app/webhook/route.ts
import { Resend } from 'resend';
import { NextRequest, NextResponse } from 'next/server';

const resend = new Resend(process.env.RESEND_API_KEY);

export async function POST(req: NextRequest) {
  try {
    const payload = await req.text();

    const event = resend.webhooks.verify({
      payload,
      headers: {
        'svix-id': req.headers.get('svix-id'),
        'svix-timestamp': req.headers.get('svix-timestamp'),
        'svix-signature': req.headers.get('svix-signature'),
      },
      secret: process.env.RESEND_WEBHOOK_SECRET,
    });

    if (event.type === 'email.received') {
      const { data: email } = await resend.emails.receiving.get(
        event.data.email_id
      );
      await processEmailForAgent(event.data, email);
    }

    return new NextResponse('OK', { status: 200 });
  } catch (error) {
    console.error('Webhook error:', error);
    return new NextResponse('Error', { status: 400 });
  }
}
```

### Register Webhook via API

```typescript
const { data, error } = await resend.webhooks.create({
  endpoint: 'https://<your-tunnel-domain>/webhook',
  events: ['email.received'],
});

// Store data.signing_secret as RESEND_WEBHOOK_SECRET
```

## Environment Variables

```bash
RESEND_API_KEY=re_xxxxxxxxx
RESEND_WEBHOOK_SECRET=whsec_xxxxxxxxx
SECURITY_LEVEL=strict
ALLOWED_SENDERS=you@email.com,trusted@example.com
ALLOWED_DOMAINS=yourcompany.com
OWNER_EMAIL=you@email.com
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| No sender verification | Always validate who sent the email before processing |
| Trusting email headers | Use webhook verification, not email headers for auth |
| Same treatment for all emails | Differentiate trusted vs untrusted senders |
| Using `express.json()` on webhook route | Use `express.raw({ type: 'application/json' })` |
| Returning non-200 for rejected emails | Always return 200 to acknowledge receipt |
| Old Resend SDK version | Check SDK version requirements above |

For the complete implementation with all 5 security levels, tunneling options, Clawdbot integration, and troubleshooting, see the full skill at https://github.com/resend/resend-skills/blob/main/agent-email-inbox/SKILL.md
