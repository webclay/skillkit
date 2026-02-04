# Better Auth OAuth / Social Login

## Configuration

Add to `lib/auth.ts`:

```ts
export const auth = betterAuth({
  // ...other config
  socialProviders: {
    google: {
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    },
    github: {
      clientId: process.env.GITHUB_CLIENT_ID!,
      clientSecret: process.env.GITHUB_CLIENT_SECRET!,
    },
    // More: apple, discord, facebook, microsoft, spotify, twitter
  },
});
```

## Environment Variables

```env
# Google OAuth
GOOGLE_CLIENT_ID=your-client-id
GOOGLE_CLIENT_SECRET=your-client-secret

# GitHub OAuth
GITHUB_CLIENT_ID=your-client-id
GITHUB_CLIENT_SECRET=your-client-secret
```

## Google OAuth Setup

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create project (or select existing)
3. Go to "APIs & Services" → "Credentials"
4. Create "OAuth 2.0 Client ID"
5. Add authorized redirect URI: `http://localhost:3000/api/auth/callback/google`
6. Copy Client ID and Secret

## GitHub OAuth Setup

1. Go to [GitHub Developer Settings](https://github.com/settings/developers)
2. Create "New OAuth App"
3. Set callback URL: `http://localhost:3000/api/auth/callback/github`
4. Copy Client ID and Secret

## Social Sign In Buttons

```tsx
"use client";

import { signIn } from "@/lib/auth-client";

export function GoogleSignInButton() {
  const handleClick = async () => {
    await signIn.social({
      provider: "google",
      callbackURL: "/dashboard",
    });
  };

  return (
    <button onClick={handleClick}>
      Sign in with Google
    </button>
  );
}

export function GitHubSignInButton() {
  const handleClick = async () => {
    await signIn.social({
      provider: "github",
      callbackURL: "/dashboard",
    });
  };

  return (
    <button onClick={handleClick}>
      Sign in with GitHub
    </button>
  );
}
```

## Custom OAuth State (v1.4+)

Pass data through OAuth flow (for referrals, attribution):

```tsx
await signIn.social({
  provider: "google",
  callbackURL: "/dashboard",
  state: {
    referralCode: "ABC123",
    source: "landing-page",
  },
});
```

Access in server hooks:

```ts
export const auth = betterAuth({
  hooks: {
    after: [{
      matcher: (ctx) => ctx.path === "/sign-in/social",
      handler: async (ctx) => {
        const { referralCode } = ctx.body.state || {};
        if (referralCode && ctx.session?.user) {
          // Store referral info
        }
      },
    }],
  },
});
```

## Account Linking

Allow users to link multiple providers:

```ts
export const auth = betterAuth({
  account: {
    accountLinking: {
      enabled: true,
      trustedProviders: ["google", "github"],
    },
  },
});
```
