---
name: resend
description: Use this skill when working with Resend for sending or receiving emails, building React Email templates, or configuring deliverability and compliance. Activate when the user mentions Resend, transactional emails, React Email, inbound webhooks, SPF/DKIM/DMARC, newsletter sending, or email compliance (GDPR, CAN-SPAM).
---

# Resend

Modern email platform for developers. Send and receive emails, build templates with React Email, handle deliverability, and ensure compliance.

## When to Use This Skill

- User asks to add email sending or receiving
- package.json contains "resend" or "@react-email/components"
- User mentions transactional email, notifications, or password reset
- User asks about React Email templates
- User wants to send welcome emails or newsletters
- User asks about email deliverability, SPF/DKIM/DMARC
- User needs email compliance (GDPR, CAN-SPAM, CASL)
- User wants to set up inbound email or webhooks
- User asks about email best practices

## Guide Index - Read the Right File for the Topic

**This skill has multiple reference files. Use this table to find the right one:**

| Topic | Read this file | What it covers |
|-------|---------------|----------------|
| **Sending emails** | [official-send-email.md](official-send-email.md) | Single/batch sending, idempotency keys, retries, webhooks, tags, domain warm-up, suppression lists, testing |
| **React Email templates** | [official-react-email.md](official-react-email.md) | Building HTML emails with React components, Tailwind CSS, static files, rendering, sending |
| **React Email components** | [official-react-components.md](official-react-components.md) | Complete component reference (Html, Body, Container, Button, Img, etc.) |
| **React Email patterns** | [official-react-patterns.md](official-react-patterns.md) | Password reset, order confirmation, notification, newsletter, team invitation templates |
| **React Email i18n** | [official-react-i18n.md](official-react-i18n.md) | Multi-language email support (next-intl, react-intl, react-i18next) |
| **React Email sending** | [official-react-sending.md](official-react-sending.md) | Sending via Resend, Nodemailer, SendGrid, or any provider |
| **Email templates (API)** | [official-templates.md](official-templates.md) | Create/update/publish/delete templates via API, variable syntax, aliases |
| **Receiving emails** | [official-inbound.md](official-inbound.md) | Inbound domain setup, webhooks, retrieving content/attachments, forwarding |
| **AI agent inbox** | [official-agent-inbox.md](official-agent-inbox.md) | Secure email for AI agents, 5 security levels, prompt injection protection |
| **Email best practices** | [official-best-practices.md](official-best-practices.md) | Router to all best practice topics below |
| **Deliverability** | [official-deliverability.md](official-deliverability.md) | SPF/DKIM/DMARC setup, sender reputation, bounce/complaint handling |
| **Compliance** | [official-compliance.md](official-compliance.md) | CAN-SPAM, GDPR, CASL requirements, unsubscribe headers, consent management |
| **Transactional emails** | [official-transactional-emails.md](official-transactional-emails.md) | Subject lines, content structure, mobile-first design, OTP codes |
| **Transactional catalog** | [official-transactional-catalog.md](official-transactional-catalog.md) | Which emails your app needs by type (SaaS, e-commerce, fintech, etc.) |
| **Marketing emails** | [official-marketing-emails.md](official-marketing-emails.md) | Opt-in requirements, content design, segmentation, frequency |
| **Email capture** | [official-email-capture.md](official-email-capture.md) | Validation, double opt-in, form design, error handling |
| **Email types** | [official-email-types.md](official-email-types.md) | Transactional vs marketing - legal distinctions, when to use each |
| **Sending reliability** | [official-sending-reliability.md](official-sending-reliability.md) | Idempotency, retry logic, error handling, queuing |
| **Webhooks & events** | [official-webhooks-events.md](official-webhooks-events.md) | Delivery events, signature verification, bounce/complaint processing |
| **List management** | [official-list-management.md](official-list-management.md) | Suppression lists, list hygiene, re-engagement, data retention |

**When in doubt:** Start with this file for quick setup, then read the specific file for the topic.

---

## Quick Start

### Step 1: Install Resend

```bash
pnpm add resend
```

For React Email templates, also install:
```bash
pnpm add @react-email/components
```

### Step 2: Set Environment Variables

```env
RESEND_API_KEY=re_...
```

### Step 3: Create Resend Client

**lib/resend.ts:**
```ts
import { Resend } from 'resend';

export const resend = new Resend(process.env.RESEND_API_KEY);
```

### Step 4: Send Email

```ts
import { resend } from '@/lib/resend';

const { data, error } = await resend.emails.send(
  {
    from: 'App <noreply@yourdomain.com>',
    to: ['user@example.com'],
    subject: 'Welcome!',
    html: '<p>Thanks for signing up!</p>',
  },
  { idempotencyKey: `welcome/${userId}` }
);

if (error) {
  console.error('Email error:', error);
  return { success: false, error: error.message };
}

return { success: true, id: data?.id };
```

### Step 5: Domain Setup (Production)

1. Go to Resend Dashboard -> Domains
2. Add your domain
3. Add DNS records (SPF, DKIM, DMARC)
4. Verify

See [official-deliverability.md](official-deliverability.md) for detailed DNS setup.

## Quick Routing

**Need to send emails?** Read [official-send-email.md](official-send-email.md)
- Single or batch, idempotency keys, retries, webhooks

**Need React Email templates?** Read [official-react-email.md](official-react-email.md)
- Component-based templates with Tailwind CSS

**Need to receive emails?** Read [official-inbound.md](official-inbound.md)
- MX records, webhooks, attachments

**Emails going to spam?** Read [official-deliverability.md](official-deliverability.md)
- SPF/DKIM/DMARC, sender reputation

**Need compliance guidance?** Read [official-compliance.md](official-compliance.md)
- CAN-SPAM, GDPR, CASL requirements

**Planning which emails to build?** Read [official-transactional-catalog.md](official-transactional-catalog.md)
- Email combinations by app type

**Setting up AI agent email?** Read [official-agent-inbox.md](official-agent-inbox.md)
- Security levels, prompt injection protection

## Testing

Use Resend's safe test addresses - never test with fake addresses at real providers:

| Address | Result |
|---------|--------|
| `delivered@resend.dev` | Simulates successful delivery |
| `bounced@resend.dev` | Simulates hard bounce |
| `complained@resend.dev` | Simulates spam complaint |

## Tips

- Use `from: 'Name <email@yourdomain.com>'` format
- Always include idempotency keys for production sends
- Use verified domain for production (not `@resend.dev`)
- Add List-Unsubscribe header for marketing emails
- Disable open/click tracking for transactional emails
- Keep email body under 102KB (Gmail clips larger)

## How to Verify

### Quick Checks
- Send test email returns `data.id` (not error)
- Check Resend Dashboard -> Emails shows sent message
- No API key errors in terminal

### Common Issues
- "Invalid API key": Check RESEND_API_KEY starts with `re_`
- "Sender not verified": Use verified domain or `@resend.dev` for testing
- Email in spam: Check DNS records (SPF, DKIM, DMARC) - see [official-deliverability.md](official-deliverability.md)
- React template errors: Check all imports from `@react-email/components`

## Official Resend Skills

These files are based on Resend's official agent skills:
- [resend/resend-skills](https://github.com/resend/resend-skills) - Sending, receiving, templates, agent inbox
- [resend/email-best-practices](https://github.com/resend/email-best-practices) - Deliverability, compliance, best practices
- [resend/react-email](https://github.com/resend/react-email/tree/canary/skills/react-email) - React Email components and patterns
