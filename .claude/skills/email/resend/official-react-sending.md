# Sending Emails with React Email

Guidelines for sending emails built with React Email.

**Important:** Use verified domains in `from` addresses. Ask the user for their verified domain.

## Send with Resend (Recommended)

```tsx
import { Resend } from 'resend';
import { WelcomeEmail } from './emails/welcome';

const resend = new Resend(process.env.RESEND_API_KEY);

const { data, error } = await resend.emails.send({
  from: 'Acme <onboarding@resend.dev>',
  to: ['user@example.com'],
  subject: 'Welcome to Acme',
  react: <WelcomeEmail name="John" verificationUrl="https://example.com/verify" />
});

if (error) {
  console.error('Failed to send:', error);
}
```

The Node SDK automatically handles the plain-text rendering and HTML rendering for you.

## Upload as Resend Template

```bash
npx react-email@latest resend setup
```

Then upload templates via the React Email UI "Resend" tab.

## Send with Other Providers

### Using render() for any provider

```tsx
import { render } from '@react-email/components';
import { WelcomeEmail } from './emails/welcome';

const html = await render(
  <WelcomeEmail name="John" verificationUrl="https://example.com/verify" />
);

const text = await render(
  <WelcomeEmail name="John" verificationUrl="https://example.com/verify" />,
  { plainText: true }
);
```

### Nodemailer

```tsx
import nodemailer from 'nodemailer';

const transporter = nodemailer.createTransport({
  host: 'smtp.example.com',
  port: 587,
  auth: { user: process.env.SMTP_USER, pass: process.env.SMTP_PASS }
});

await transporter.sendMail({
  from: 'noreply@example.com',
  to: 'user@example.com',
  subject: 'Welcome',
  html
});
```

### SendGrid

```tsx
import sgMail from '@sendgrid/mail';

sgMail.setApiKey(process.env.SENDGRID_API_KEY);

await sgMail.send({
  to: 'user@example.com',
  from: 'noreply@example.com',
  subject: 'Welcome',
  html
});
```
