---
name: unosend
description: Use this skill when sending transactional or marketing emails with Unosend. Activate when the user mentions Unosend, email sending API, email validation, scheduled emails, bulk email, or audience/contact management via Unosend.
---

# Unosend

Email, SMS, and WhatsApp API platform with competitive pricing and high limits. Supports transactional email, bulk sending, templates, email validation, audiences, contacts, webhooks, and scheduled delivery.

## When to Use This Skill

- User asks to add email sending and prefers Unosend
- package.json contains "unosend"
- User mentions transactional email, notifications, or password reset with Unosend
- User wants bulk email sending via Unosend
- User wants email validation (syntax, MX, disposable, catch-all detection)
- User asks about audience/contact management with Unosend
- User wants scheduled email delivery

## Instructions

### Step 1: Install Unosend SDK

```bash
[pm] add unosend
```

Requires Node.js 18+. Full TypeScript support included.

### Step 2: Set Environment Variables

```env
UNOSEND_API_KEY=un_xxxxxxxxxx
```

API keys are prefixed with `un_` and shown only once on creation. Use different keys for development and production.

**Key permission levels:**
- Full Access - read and write all resources
- Read Only - view only
- Send Only - email sending only

### Step 3: Create Unosend Client

**lib/unosend.ts:**
```ts
import { Unosend } from 'unosend';

export const unosend = new Unosend(process.env.UNOSEND_API_KEY!);
```

**Client options (optional):**
```ts
export const unosend = new Unosend(process.env.UNOSEND_API_KEY!, {
  timeout: 30000,    // ms (default)
});
```

### Step 4: Send Email

```ts
import { unosend } from '@/lib/unosend';

const { data, error } = await unosend.emails.send({
  from: 'App Name <noreply@yourdomain.com>',
  to: ['user@example.com'],
  subject: 'Welcome!',
  html: '<h1>Welcome!</h1><p>Thanks for signing up.</p>',
});

if (error) {
  console.error('Email error:', error);
  return { success: false, error: error.message };
}

return { success: true, id: data?.id };
```

## Examples

**Plain Text Email:**
```ts
await unosend.emails.send({
  from: 'Team <team@yourdomain.com>',
  to: ['user@example.com'],
  subject: 'Hello',
  text: 'Plain text version of the email.',
});
```

**HTML Email with CC/BCC and Reply-To:**
```ts
await unosend.emails.send({
  from: 'Team <team@yourdomain.com>',
  to: ['customer@example.com'],
  cc: ['manager@yourdomain.com'],
  bcc: ['archive@yourdomain.com'],
  reply_to: 'support@yourdomain.com',
  subject: 'Order Update',
  html: '<h1>Your order shipped!</h1>',
});
```

**With Template and Variables:**
```ts
await unosend.emails.send({
  from: 'noreply@yourdomain.com',
  to: ['user@example.com'],
  subject: 'Order Confirmation',
  template_id: 'tpl_abc123',
  template_data: {
    first_name: 'John',
    order_number: '#12345',
    order_total: '$99.00',
  },
});
```

Template variables use `{{variable}}` mustache syntax in subject, HTML, and text fields.

**With Attachments:**
```ts
await unosend.emails.send({
  from: 'billing@yourdomain.com',
  to: ['customer@example.com'],
  subject: 'Your Invoice',
  html: '<p>Please find your invoice attached.</p>',
  attachments: [
    {
      filename: 'invoice.pdf',
      content: base64EncodedContent,
      content_type: 'application/pdf',
    },
  ],
});
```

**Scheduled Email:**
```ts
await unosend.emails.send({
  from: 'noreply@yourdomain.com',
  to: ['user@example.com'],
  subject: 'Reminder',
  html: '<p>Your trial ends tomorrow.</p>',
  scheduled_for: '2026-03-15T09:00:00Z', // ISO 8601
});
```

**Priority Levels:**
```ts
await unosend.emails.send({
  from: 'noreply@yourdomain.com',
  to: ['user@example.com'],
  subject: 'Password Reset',
  html: '<p>Reset your password: <a href="...">Click here</a></p>',
  priority: 'high', // 'high' (transactional), 'normal' (default), 'low' (bulk)
});
```

**With Tags and Tracking:**
```ts
await unosend.emails.send({
  from: 'marketing@yourdomain.com',
  to: ['user@example.com'],
  subject: 'Weekly Newsletter',
  html: '<p>This week in updates...</p>',
  tags: [{ name: 'campaign', value: 'weekly-newsletter' }],
  tracking: { open: true, click: true }, // default: both true
});
```

**Batch Email (up to 100):**
```ts
await unosend.emails.sendBatch([
  {
    from: 'hello@yourdomain.com',
    to: ['user1@example.com'],
    subject: 'Hello User 1',
    html: '<p>Welcome!</p>',
  },
  {
    from: 'hello@yourdomain.com',
    to: ['user2@example.com'],
    subject: 'Hello User 2',
    html: '<p>Welcome!</p>',
    tracking: { open: false, click: false },
  },
]);
```

Each email in a batch can have its own tracking settings.

**Get Email Status:**
```ts
const { data } = await unosend.emails.get('email_id');
// data.status: 'queued' | 'sent' | 'delivered' | 'bounced' | 'complained'
```

**List Emails:**
```ts
const { data } = await unosend.emails.list();
```

**Error Handling:**
```ts
const { data, error } = await unosend.emails.send({ ... });

if (error) {
  // error.statusCode: 401 (invalid key), 422 (validation), 429 (rate limit)
  // error.retryAfter: seconds to wait (on 429)
  console.error('Email failed:', error.statusCode, error.message);
  return { success: false };
}

return { success: true, id: data.id };
```

## Better Auth Integration

Unosend works with Better Auth for verification emails, password resets, and OTPs.

**Environment variables:**
```env
UNOSEND_API_KEY=un_xxxxxxxxxx
FROM_EMAIL=noreply@yourdomain.com
FROM_NAME=Your App Name
```

**Email helper (lib/email.ts):**
```ts
import { Unosend } from 'unosend';

const unosend = new Unosend(process.env.UNOSEND_API_KEY!);
const FROM_EMAIL = process.env.FROM_EMAIL || 'noreply@yourdomain.com';
const FROM_NAME = process.env.FROM_NAME || 'Your App';

interface SendEmailOptions {
  to: string;
  templateId: string;
  templateData: Record<string, string>;
}

export async function sendEmail({ to, templateId, templateData }: SendEmailOptions) {
  const { data, error } = await unosend.emails.send({
    from: `${FROM_NAME} <${FROM_EMAIL}>`,
    to: [to],
    template_id: templateId,
    template_data: templateData,
  });

  if (error) {
    throw new Error(error.message || 'Failed to send email');
  }

  return data;
}
```

**Better Auth config:**
```ts
import { betterAuth } from 'better-auth';
import { emailOTP } from 'better-auth/plugins';
import { sendEmail } from './email';

export const auth = betterAuth({
  emailVerification: {
    sendOnSignUp: true,
    sendVerificationEmail: async ({ user, url }) => {
      sendEmail({
        to: user.email,
        templateId: 'tpl_verify_email',
        templateData: {
          userName: user.name || 'there',
          verificationUrl: url,
        },
      });
    },
  },
  emailAndPassword: {
    enabled: true,
    sendResetPassword: async ({ user, url }) => {
      sendEmail({
        to: user.email,
        templateId: 'tpl_password_reset',
        templateData: {
          userName: user.name || 'there',
          resetUrl: url,
        },
      });
    },
  },
  plugins: [
    emailOTP({
      sendVerificationOTP: async ({ email, otp, type }) => {
        const templateIds: Record<string, string> = {
          'sign-in': 'tpl_otp_signin',
          'email-verification': 'tpl_otp_verify',
          'forget-password': 'tpl_otp_reset',
        };
        sendEmail({
          to: email,
          templateId: templateIds[type],
          templateData: { otp },
        });
      },
    }),
  ],
});
```

**Required templates to create in Unosend dashboard:**

| Template | ID | Variables |
|----------|----|-----------|
| Email verification | `tpl_verify_email` | `{{userName}}`, `{{verificationUrl}}` |
| Password reset | `tpl_password_reset` | `{{userName}}`, `{{resetUrl}}` |
| Sign-in OTP | `tpl_otp_signin` | `{{otp}}` |
| Verification OTP | `tpl_otp_verify` | `{{otp}}` |
| Password reset OTP | `tpl_otp_reset` | `{{otp}}` |

## Template Management

**Create template:**
```ts
const { data } = await fetch('https://www.unosend.co/api/v1/templates', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${process.env.UNOSEND_API_KEY}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    name: 'Welcome Email',
    subject: 'Welcome, {{first_name}}!',
    html: '<h1>Welcome, {{first_name}}!</h1><p>Thanks for joining {{company_name}}.</p>',
    text: 'Welcome, {{first_name}}! Thanks for joining {{company_name}}.',
  }),
}).then(r => r.json());
// Returns: { id: 'tpl_abc123', name, subject, html, text, created_at }
```

## Audience and Contact Management

**Create audience:**
```ts
const { data } = await unosend.audiences.create({ name: 'Newsletter Subscribers' });
// Returns: { id: 'aud_123', name, created_at }
```

**Add contact to audience:**
```ts
await unosend.contacts.create('aud_123', {
  email: 'user@example.com',
  first_name: 'John',
  last_name: 'Doe',
  unsubscribed: false,
});
```

**List contacts:**
```ts
const { data } = await unosend.contacts.list('aud_123');
```

## Email Validation

Validate email addresses before sending to reduce bounces.

**Single validation:**
```ts
const result = await unosend.validateEmail({
  email: 'user@example.com',
  check_smtp: true,      // check if mailbox exists (optional)
  check_catch_all: true,  // detect catch-all domains (optional)
});

// result: {
//   email: 'user@example.com',
//   valid: true,
//   score: 90,            // 0-100 quality score
//   reason: null,
//   checks: {
//     syntax: true,
//     mx_records: true,
//     disposable: false,
//     role_based: false,
//     free_provider: true,
//     catch_all: false,
//     smtp_valid: true,
//   },
//   details: {
//     local_part: 'user',
//     domain: 'example.com',
//     mx_hosts: ['mx1.example.com'],
//     suggestion: null,    // typo correction (e.g., 'gmial.com' -> 'gmail.com')
//   },
// }
```

Bulk validation supports up to 1 million addresses.

## Webhooks

Track email events with webhooks.

**Supported events:**
- `email.sent` - email accepted by mail server
- `email.delivered` - email delivered to inbox
- `email.opened` - recipient opened email
- `email.clicked` - recipient clicked a link
- `email.bounced` - email bounced
- `email.complained` - recipient marked as spam

**Create webhook:**
```ts
const { data } = await unosend.webhooks.create({
  url: 'https://yourdomain.com/api/webhooks/unosend',
  events: ['email.delivered', 'email.bounced', 'email.complained'],
});
// data.signing_secret - store securely, shown only once
```

Store `signing_secret` securely for signature verification. It is only shown once.

## Domain Setup

For production, verify your sending domain:

1. Go to Unosend Dashboard or use the API
2. Add your domain
3. Add DNS records returned by the API:
   - TXT record for domain verification
   - TXT record for SPF authentication
   - CNAME record for DKIM signing
4. Call the verify endpoint

**API domain creation:**
```ts
await unosend.domains.create({ domain: 'mail.yourdomain.com' });
// Returns domain ID + DNS records to add
```

**Verify domain:**
```ts
await unosend.domains.verify('domain_id');
```

Rate limit: 10 domain creations per hour per organization.

## API Reference

**Base URL:** `https://www.unosend.co/api/v1`

**Auth header:** `Authorization: Bearer un_xxxxxxxxxx`

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/emails` | POST | Send single email |
| `/emails/batch` | POST | Send batch email (max 100) |
| `/emails/{id}` | GET | Get email status |
| `/templates` | POST | Create template |
| `/domains` | POST | Add domain |
| `/domains/{id}/verify` | POST | Verify domain |
| `/audiences` | POST | Create audience |
| `/audiences/{id}/contacts` | POST | Add contact |
| `/webhooks` | POST | Create webhook |
| `/validate/email` | POST | Validate single email |

**Error codes:**
- 401 - Invalid or missing API key
- 403 - Insufficient permissions
- 422 - Validation error
- 429 - Rate limited (check `retryAfter`)

## TypeScript Types

The SDK exports useful types:

```ts
import type { SendEmailRequest, Email, Domain, Contact, UnosendError } from 'unosend';
```

## Tips

- Use `from: 'Name <email@yourdomain.com>'` format (string, not object)
- `to` is always an array of strings
- Use verified domain for production
- Template variables use `{{variableName}}` mustache syntax
- Set `priority: 'high'` for transactional emails (password resets, OTPs)
- Set `priority: 'low'` for bulk/marketing emails
- Disable tracking for transactional emails: `tracking: { open: false, click: false }`
- Batch operations limited to 100 emails per request
- Free tier: 5,000 emails/month
- Pro and Enterprise plans have no rate limiting

## How to Verify

### Quick Checks
- Send test email returns `data.id` (not error)
- Check Unosend Dashboard shows sent message
- No API key errors in terminal

### Manual Verification
- Trigger email from your app (signup, form submission, etc.)
- Check recipient inbox (or spam folder) for email
- Verify email content and styling looks correct
- Check email status via `unosend.emails.get(id)` returns `delivered`

### Common Issues
- "Unauthorized" (401): Check API key starts with `un_` and is valid
- "Insufficient permissions" (403): Check API key permission level
- "Sender not verified": Verify domain in Unosend Dashboard
- Email in spam: Check DNS records (SPF, DKIM, DMARC)
- Rate limited (429): Check `error.retryAfter` for wait time
- Template variable not rendering: Check `template_data` keys match `{{variable}}` names
