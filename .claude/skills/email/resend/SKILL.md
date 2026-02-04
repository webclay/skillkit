---
name: resend
description: Resend email service for sending transactional and marketing emails. Trigger words - resend, email, send email, transactional email, notification email, react email, email template, password reset email
---

# Resend

Modern email API with React Email support for building beautiful, reliable transactional emails.

## When to Use This Skill

- User asks to add email sending
- package.json contains "resend"
- User mentions transactional email, notifications, or password reset
- User asks about React Email templates
- User wants to send welcome emails or newsletters

## Instructions

### Step 1: Install Resend

```bash
pnpm add resend @react-email/components
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

const { data, error } = await resend.emails.send({
  from: 'App <noreply@yourdomain.com>',
  to: ['user@example.com'],
  subject: 'Welcome!',
  html: '<p>Thanks for signing up!</p>',
});
```

## Examples

**Basic Email:**
```ts
await resend.emails.send({
  from: 'Team <team@yourdomain.com>',
  to: ['user@example.com'],
  subject: 'Hello',
  text: 'Plain text version',
  html: '<p>HTML version</p>',
});
```

**React Email Template:**
```tsx
// emails/WelcomeEmail.tsx
import { Html, Body, Container, Heading, Text, Button } from '@react-email/components';

export function WelcomeEmail({ name, url }: { name: string; url: string }) {
  return (
    <Html>
      <Body style={{ fontFamily: 'sans-serif' }}>
        <Container>
          <Heading>Welcome, {name}!</Heading>
          <Text>Thanks for joining us.</Text>
          <Button href={url}>Get Started</Button>
        </Container>
      </Body>
    </Html>
  );
}
```

**Send React Email:**
```ts
import { WelcomeEmail } from '@/emails/WelcomeEmail';

await resend.emails.send({
  from: 'App <noreply@yourdomain.com>',
  to: ['user@example.com'],
  subject: 'Welcome!',
  react: WelcomeEmail({ name: 'John', url: 'https://app.com/login' }),
});
```

**With Attachments:**
```ts
await resend.emails.send({
  from: 'Billing <billing@yourdomain.com>',
  to: ['user@example.com'],
  subject: 'Your Invoice',
  html: '<p>Invoice attached</p>',
  attachments: [{ filename: 'invoice.pdf', content: pdfBuffer }],
});
```

**CC/BCC:**
```ts
await resend.emails.send({
  from: 'Team <team@yourdomain.com>',
  to: ['customer@example.com'],
  cc: ['manager@yourdomain.com'],
  bcc: ['archive@yourdomain.com'],
  subject: 'Update',
  html: '<p>Content</p>',
});
```

**Error Handling:**
```ts
const { data, error } = await resend.emails.send({ ... });

if (error) {
  console.error('Email error:', error);
  return { success: false, error: error.message };
}

return { success: true, id: data?.id };
```

## Reference Files

- [templates.md](templates.md) - Email template patterns

## Domain Setup

For production, verify your domain:
1. Go to Resend Dashboard → Domains
2. Add your domain
3. Add DNS records (SPF, DKIM)
4. Verify

## Tips

- Use `from: 'Name <email@yourdomain.com>'` format
- Preview templates locally: `pnpm email:dev`
- Use verified domain for production (not `@resend.dev`)
- Add List-Unsubscribe header for marketing emails

## How to Verify

### Quick Checks
- Send test email returns `data.id` (not error)
- Check Resend Dashboard → Emails shows sent message
- No API key errors in terminal

### Manual Verification
- Trigger email from your app (signup, form submission, etc.)
- Check recipient inbox (or spam folder) for email
- Verify email content and styling looks correct
- Test on multiple email clients if design matters

### Common Issues
- "Invalid API key": Check RESEND_API_KEY starts with `re_`
- "Sender not verified": Use verified domain or `@resend.dev` for testing
- Email in spam: Verify domain DNS records (SPF, DKIM)
- React template errors: Check all imports from `@react-email/components`
