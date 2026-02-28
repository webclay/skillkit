# Common Email Patterns

Real-world examples of common email templates using React Email with Tailwind CSS styling.

## Password Reset Email

```tsx
import {
  Html,
  Head,
  Preview,
  Body,
  Container,
  Heading,
  Text,
  Button,
  Hr,
  Tailwind,
  pixelBasedPreset
} from '@react-email/components';

interface PasswordResetProps {
  resetUrl: string;
  email: string;
  expiryHours?: number;
}

export default function PasswordReset({ resetUrl, email, expiryHours = 1 }: PasswordResetProps) {
  return (
    <Html lang="en">
      <Tailwind config={{ presets: [pixelBasedPreset] }}>
        <Head />
        <Preview>Reset your password - Action required</Preview>
        <Body className="bg-gray-100 font-sans">
          <Container className="mx-auto py-10 px-5 max-w-xl bg-white">
            <Heading className="text-2xl font-bold text-gray-800 mb-5">
              Reset Your Password
            </Heading>
            <Text className="text-base leading-7 text-gray-800 my-4">
              A password reset was requested for your account: <strong>{email}</strong>
            </Text>
            <Text className="text-base leading-7 text-gray-800 my-4">
              Click the button below to reset your password. This link expires in {expiryHours} hour{expiryHours > 1 ? 's' : ''}.
            </Text>
            <Button
              href={resetUrl}
              className="bg-red-600 text-white px-7 py-3.5 rounded block text-center font-bold my-6 no-underline"
            >
              Reset Password
            </Button>
            <Hr className="border-gray-200 my-6" />
            <Text className="text-sm text-gray-500 leading-5 my-2">
              If you didn't request this, please ignore this email. Your password will remain unchanged.
            </Text>
            <Text className="text-sm text-gray-500 leading-5 my-2">
              For security, this link will only work once.
            </Text>
          </Container>
        </Body>
      </Tailwind>
    </Html>
  );
}

PasswordReset.PreviewProps = {
  resetUrl: 'https://example.com/reset/abc123',
  email: 'user@example.com',
  expiryHours: 1
} as PasswordResetProps;
```

## Team Invitation Email

```tsx
import {
  Html,
  Head,
  Preview,
  Body,
  Container,
  Section,
  Heading,
  Text,
  Button,
  Hr,
  Tailwind,
  pixelBasedPreset
} from '@react-email/components';

interface TeamInvitationProps {
  inviterName: string;
  inviterEmail: string;
  teamName: string;
  role: string;
  inviteUrl: string;
  expiryDays: number;
}

export default function TeamInvitation({
  inviterName,
  inviterEmail,
  teamName,
  role,
  inviteUrl,
  expiryDays
}: TeamInvitationProps) {
  return (
    <Html lang="en">
      <Tailwind config={{ presets: [pixelBasedPreset] }}>
        <Head />
        <Preview>You've been invited to join {teamName}</Preview>
        <Body className="bg-gray-100 font-sans">
          <Container className="mx-auto py-10 px-5 max-w-xl bg-white">
            <Heading className="text-3xl font-bold text-gray-800 text-center mb-6">
              You're Invited!
            </Heading>

            <Text className="text-base leading-7 text-gray-800 my-4">
              <strong>{inviterName}</strong> ({inviterEmail}) has invited you to join the{' '}
              <strong>{teamName}</strong> team.
            </Text>

            <Section className="bg-gray-50 p-5 rounded border border-gray-200 my-6">
              <Text className="text-xs text-gray-500 uppercase font-bold mb-2">Role</Text>
              <Text className="text-lg font-bold text-gray-800 m-0">{role}</Text>
            </Section>

            <Text className="text-base leading-7 text-gray-800 my-4">
              Click the button below to accept the invitation and get started.
            </Text>

            <Button
              href={inviteUrl}
              className="bg-green-600 text-white px-7 py-3.5 rounded block text-center font-bold text-base my-6 no-underline"
            >
              Accept Invitation
            </Button>

            <Hr className="border-gray-200 my-6" />

            <Text className="text-sm text-gray-500 leading-5 my-2">
              This invitation will expire in {expiryDays} day{expiryDays > 1 ? 's' : ''}.
            </Text>
            <Text className="text-sm text-gray-500 leading-5 my-2">
              If you weren't expecting this invitation, you can safely ignore this email.
            </Text>
          </Container>
        </Body>
      </Tailwind>
    </Html>
  );
}

TeamInvitation.PreviewProps = {
  inviterName: 'John Doe',
  inviterEmail: 'john@example.com',
  teamName: 'Acme Corp Engineering',
  role: 'Developer',
  inviteUrl: 'https://example.com/invite/abc123',
  expiryDays: 7
} as TeamInvitationProps;
```

These patterns demonstrate:
- Tailwind CSS utility classes for styling
- Proper component usage with `pixelBasedPreset`
- TypeScript typing
- Preview props for testing
- Responsive layouts
- Common email scenarios
