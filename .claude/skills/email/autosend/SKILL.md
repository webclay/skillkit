---
name: autosend
description: AutoSend email service for transactional and marketing emails. Trigger words - autosend, auto send, email, send email, transactional email, notification email, email template, password reset email, marketing email, email campaign, bulk email
---

# AutoSend

Email platform for sending transactional and marketing emails via API or SMTP. Volume-based pricing (no contact limits) - cheaper alternative to Resend, Postmark, and SendGrid.

## When to Use This Skill

- User asks to add email sending and prefers AutoSend
- package.json contains "autosendjs"
- User mentions transactional email, notifications, or password reset
- User wants marketing emails, campaigns, or automation
- User wants bulk email sending
- User asks about email templates with dynamic variables

## Instructions

### Step 1: Install AutoSend SDK

```bash
[pm] add autosendjs
```

### Step 2: Set Environment Variables

```env
AUTOSEND_API_KEY=AS_xxxxxxxxxxxx
```

### Step 3: Create AutoSend Client

**lib/autosend.ts:**
```ts
import { Autosend } from 'autosendjs';

export const autosend = new Autosend(process.env.AUTOSEND_API_KEY!);
```

**Client options (optional):**
```ts
export const autosend = new Autosend(process.env.AUTOSEND_API_KEY!, {
  timeout: 30000,    // ms (default)
  maxRetries: 3,     // retry attempts (default)
  debug: false,      // enable debug logging
});
```

### Step 4: Send Email

```ts
import { autosend } from '@/lib/autosend';

const { data, error } = await autosend.emails.send({
  from: { email: 'noreply@yourdomain.com', name: 'App Name' },
  to: { email: 'user@example.com', name: 'John Doe' },
  subject: 'Welcome!',
  html: '<h1>Welcome!</h1><p>Thanks for signing up.</p>',
});
```

## Examples

**Plain Text Email:**
```ts
await autosend.emails.send({
  from: { email: 'team@yourdomain.com', name: 'Team' },
  to: { email: 'user@example.com' },
  subject: 'Hello',
  text: 'Plain text version',
});
```

**HTML Email:**
```ts
await autosend.emails.send({
  from: { email: 'team@yourdomain.com', name: 'Team' },
  to: { email: 'user@example.com', name: 'John' },
  subject: 'Welcome',
  html: '<h1>Welcome!</h1><p>HTML version</p>',
});
```

**With Template and Dynamic Data:**
```ts
await autosend.emails.send({
  from: { email: 'noreply@yourdomain.com' },
  to: { email: 'user@example.com' },
  subject: 'Order Confirmation',
  templateId: 'your_template_id',
  dynamicData: {
    name: 'John',
    orderNumber: '#12345',
    orderTotal: '$99.00',
  },
});
```

Template variables use Handlebars syntax: `{{name}}`, `{{orderNumber}}`. Nested variables supported: `{{order.total}}`.

**With CC/BCC and Reply-To:**
```ts
await autosend.emails.send({
  from: { email: 'team@yourdomain.com', name: 'Team' },
  to: { email: 'customer@example.com' },
  cc: [{ email: 'manager@yourdomain.com' }],
  bcc: [{ email: 'archive@yourdomain.com' }],
  replyTo: { email: 'support@yourdomain.com' },
  subject: 'Update',
  html: '<p>Content</p>',
});
```

**Bulk Email (up to 100 recipients):**
```ts
await autosend.emails.bulk({
  emails: [
    {
      from: { email: 'hello@yourdomain.com' },
      to: { email: 'user1@example.com' },
      subject: 'Hello User 1',
      html: '<p>Welcome!</p>',
    },
    {
      from: { email: 'hello@yourdomain.com' },
      to: { email: 'user2@example.com' },
      subject: 'Hello User 2',
      html: '<p>Welcome!</p>',
    },
  ],
});
```

**Error Handling:**
```ts
try {
  const result = await autosend.emails.send({ ... });
  if (result.data) {
    return { success: true, id: result.data.emailId };
  }
} catch (error) {
  console.error('Email error:', error);
  return { success: false, error: error.message };
}
```

## Better Auth Integration

AutoSend works with Better Auth for verification emails, password resets, and OTPs.

**Environment variables:**
```env
AUTOSEND_API_KEY=AS_xxxxxxxxxxxx
FROM_EMAIL=noreply@yourdomain.com
FROM_NAME=Your App Name
```

**Email helper (lib/email.ts):**
```ts
const AUTOSEND_API_KEY = process.env.AUTOSEND_API_KEY!;
const FROM_EMAIL = process.env.FROM_EMAIL || 'noreply@yourdomain.com';
const FROM_NAME = process.env.FROM_NAME || 'Your App';

interface SendEmailOptions {
  to: string;
  templateId: string;
  dynamicData: Record<string, string>;
}

export async function sendEmail({ to, templateId, dynamicData }: SendEmailOptions) {
  const response = await fetch('https://api.autosend.com/v1/mails/send', {
    method: 'POST',
    headers: {
      Authorization: `Bearer ${AUTOSEND_API_KEY}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      from: { email: FROM_EMAIL, name: FROM_NAME },
      to: { email: to },
      templateId,
      dynamicData,
    }),
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.message || 'Failed to send email');
  }

  return response.json();
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
        templateId: 'tmpl_verify_email',
        dynamicData: {
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
        templateId: 'tmpl_password_reset',
        dynamicData: {
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
          'sign-in': 'tmpl_otp_signin',
          'email-verification': 'tmpl_otp_verify',
          'forget-password': 'tmpl_otp_reset',
        };
        sendEmail({
          to: email,
          templateId: templateIds[type],
          dynamicData: { otp },
        });
      },
    }),
  ],
});
```

**Required templates to create in AutoSend dashboard:**

| Template | ID | Variables |
|----------|----|-----------|
| Email verification | `tmpl_verify_email` | `{{userName}}`, `{{verificationUrl}}` |
| Password reset | `tmpl_password_reset` | `{{userName}}`, `{{resetUrl}}` |
| Sign-in OTP | `tmpl_otp_signin` | `{{otp}}` |
| Verification OTP | `tmpl_otp_verify` | `{{otp}}` |
| Password reset OTP | `tmpl_otp_reset` | `{{otp}}` |

## Contact Management

**Create contact:**
```ts
await autosend.contacts.create({
  email: 'user@example.com',
  firstName: 'John',
  lastName: 'Doe',
  listIds: ['list_abc123'],
  customFields: { company: 'Acme Inc', plan: 'pro' },
});
```

**Upsert contact (create or update):**
```ts
await autosend.contacts.upsert({
  email: 'user@example.com',
  firstName: 'Jane',
  customFields: { plan: 'enterprise' },
});
```

**Get and delete:**
```ts
const contact = await autosend.contacts.get('contact_id');
await autosend.contacts.delete('contact_id');
```

## Resend Compatibility

AutoSend provides a drop-in Resend adapter for migrating existing projects:

```ts
import { Resend } from 'autosendjs/resend';

const resend = new Resend('AS_xxxxxxxxxxxx');
// Or uses RESEND_API_KEY env var automatically:
const resend = new Resend();
```

This lets you switch from Resend to AutoSend without changing existing email code.

## SMTP Alternative

For platforms that need SMTP instead of API (WordPress, Supabase Auth, Auth0):

| Setting | Value |
|---------|-------|
| Host | `smtp.autosend.com` |
| Port | `465` (TLS) or `587` (STARTTLS) |
| Username | `autosend` |
| Password | Your API key (`AS_xxxx`) |
| Encryption | TLS/SSL |

## Webhooks

Track email events (delivered, opened, clicked, bounced) with webhooks.

**Webhook signature verification:**
```ts
import crypto from 'crypto';

function verifyWebhook(body: string, signature: string, secret: string): boolean {
  const expected = crypto
    .createHmac('sha256', secret)
    .update(body)
    .digest('hex');
  return signature === expected;
}
```

Signature is in the `X-Webhook-Signature` header. Webhooks retry up to 3 times with exponential backoff (1 min, 5 min, 15 min).

## Domain Setup

For production, verify your sending domain:
1. Go to AutoSend Dashboard > Settings > Domains
2. Add your domain (recommended: subdomain like `mail.yourdomain.com`)
3. Add DNS records (SPF, DKIM, DMARC)
4. Verify

## API Reference

**Base URL:** `https://api.autosend.com`

**Auth header:** `Authorization: Bearer AS_xxxxxxxxxxxx`

**Rate limits:** 2 requests/second or 50 requests/minute per API key.

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/v1/mails/send` | POST | Send single email |
| `/v1/mails/bulk` | POST | Send bulk email (max 100) |
| `/v1/contacts` | POST | Create contact |
| `/v1/contacts/email` | POST | Upsert contact |
| `/v1/contacts/bulk-update` | POST | Bulk update contacts (max 100) |
| `/v1/contacts/{id}` | GET | Get contact |
| `/v1/contacts/{id}` | DELETE | Delete contact |
| `/v1/contacts/search/emails` | POST | Search contacts by email |
| `/v1/contacts/remove` | POST | Remove contacts by email |

## Tips

- Use `from: { email: 'name@yourdomain.com', name: 'Display Name' }` format
- Use verified domain for production (not test domains)
- Template variables use `{{variableName}}` Handlebars syntax
- Nested variables supported: `{{user.firstName}}` (transactional only, not campaigns)
- Conditional logic: `{{#if firstName}}{{firstName}}{{else}}there{{/if}}`
- Add `unsubscribeGroupId` for marketing emails (required for compliance)
- Bulk operations limited to 100 recipients/contacts per request

## How to Verify

### Quick Checks
- Send test email returns `data.emailId` (not error)
- Check AutoSend Dashboard > Email Activity shows sent message
- No API key errors in terminal

### Manual Verification
- Trigger email from your app (signup, form submission, etc.)
- Check recipient inbox (or spam folder) for email
- Verify email content and styling looks correct

### Common Issues
- "Unauthorized": Check API key starts with `AS_` and is valid
- "Sender not verified": Verify domain in AutoSend Dashboard
- Email in spam: Check DNS records (SPF, DKIM, DMARC)
- Rate limited (429): Implement exponential backoff, max 2 req/s
- Template variable not rendering: Check `dynamicData` keys match `{{variable}}` names
