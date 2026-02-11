# Environment Variables Template

Reference for generating `.env` and `.env.production` files during project setup.

## Framework Prefix Map

Variables marked `[CLIENT]` are exposed to the browser. Replace `[CLIENT]` with the correct framework prefix:

| Framework | Client Prefix | Example |
|-----------|--------------|---------|
| TanStack Start | `VITE_` | `VITE_APP_URL` |
| Next.js | `NEXT_PUBLIC_` | `NEXT_PUBLIC_APP_URL` |
| Astro | `PUBLIC_` | `PUBLIC_APP_URL` |
| Expo | `EXPO_PUBLIC_` | `EXPO_PUBLIC_APP_URL` |

If the framework is unknown, omit the prefix and use the base name (e.g., `APP_URL`).

---

## Service Groups

Include a group when the user selected that service during the questionnaire. Each entry shows:

- **Variable** - Name (with `[CLIENT]` marker if browser-exposed)
- **Dev Default** - Value for `.env` (empty string if none)
- **Production Comment** - Comment line above the variable in `.env.production`

---

### App (always include)

| Variable | Dev Default | Production Comment |
|----------|-------------|-------------------|
| `[CLIENT] APP_URL` | `http://localhost:3000` | `# Your production domain (e.g., https://yourapp.com)` |

---

### Database: Supabase

Include when: Supabase selected for database.

| Variable | Dev Default | Production Comment |
|----------|-------------|-------------------|
| `DATABASE_URL` | | `# Supabase Dashboard > Project Settings > Database > Connection string` |
| `SUPABASE_SERVICE_ROLE_KEY` | | `# KEEP SECRET - Supabase Dashboard > Settings > API > service_role key` |
| `[CLIENT] SUPABASE_URL` | | `# Supabase Dashboard > Settings > API > Project URL` |
| `[CLIENT] SUPABASE_ANON_KEY` | | `# Supabase Dashboard > Settings > API > anon/public key` |

---

### Database: Drizzle / Generic PostgreSQL

Include when: Drizzle selected without Supabase (standalone Postgres).

| Variable | Dev Default | Production Comment |
|----------|-------------|-------------------|
| `DATABASE_URL` | | `# Your PostgreSQL connection string` |

---

### Database: Prisma

Include when: Prisma selected without Supabase.

| Variable | Dev Default | Production Comment |
|----------|-------------|-------------------|
| `DATABASE_URL` | | `# Your PostgreSQL connection string (pooled)` |
| `DIRECT_URL` | | `# Direct connection for migrations (non-pooled)` |

---

### Database: Neon

Include when: Neon selected as database host.

| Variable | Dev Default | Production Comment |
|----------|-------------|-------------------|
| `DATABASE_URL` | | `# Neon Console > Connection Details (pooled endpoint)` |
| `DATABASE_URL_UNPOOLED` | | `# Neon Console > Connection Details (direct, for migrations)` |

---

### Database: Convex

Include when: Convex selected.

| Variable | Dev Default | Production Comment |
|----------|-------------|-------------------|
| `[CLIENT] CONVEX_URL` | | `# Convex Dashboard > Deployment URL` |

---

### Auth: Better Auth

Include when: Better Auth selected.

| Variable | Dev Default | Production Comment |
|----------|-------------|-------------------|
| `BETTER_AUTH_SECRET` | | `# Generate with: openssl rand -base64 32` |
| `BETTER_AUTH_URL` | `http://localhost:3000` | `# Same as your production domain` |

Note: `DATABASE_URL` is also required but should already be included from the database group. Do not duplicate it.

---

### Auth: Google OAuth

Include when: Google OAuth selected as a provider.

| Variable | Dev Default | Production Comment |
|----------|-------------|-------------------|
| `GOOGLE_CLIENT_ID` | | `# Google Cloud Console > APIs & Services > Credentials > OAuth 2.0` |
| `GOOGLE_CLIENT_SECRET` | | `# KEEP SECRET - Google Cloud Console > Credentials` |

---

### Auth: GitHub OAuth

Include when: GitHub OAuth selected as a provider.

| Variable | Dev Default | Production Comment |
|----------|-------------|-------------------|
| `GITHUB_CLIENT_ID` | | `# GitHub > Settings > Developer Settings > OAuth Apps` |
| `GITHUB_CLIENT_SECRET` | | `# KEEP SECRET - GitHub > Developer Settings > OAuth Apps` |

---

### Payments: Stripe

Include when: Stripe selected.

| Variable | Dev Default | Production Comment |
|----------|-------------|-------------------|
| `STRIPE_SECRET_KEY` | | `# KEEP SECRET - Stripe Dashboard > Developers > API keys > Secret key` |
| `[CLIENT] STRIPE_PUBLISHABLE_KEY` | | `# Stripe Dashboard > Developers > API keys > Publishable key` |
| `STRIPE_WEBHOOK_SECRET` | | `# KEEP SECRET - Stripe Dashboard > Developers > Webhooks` |

---

### Payments: Polar

Include when: Polar selected.

| Variable | Dev Default | Production Comment |
|----------|-------------|-------------------|
| `POLAR_API_KEY` | | `# KEEP SECRET - Polar Dashboard > Settings > API Keys` |
| `POLAR_ORGANIZATION_ID` | | `# Polar Dashboard > Organization > Settings` |
| `POLAR_WEBHOOK_SECRET` | | `# KEEP SECRET - Polar Dashboard > Webhooks > Secret` |

---

### Email: Resend

Include when: Resend selected.

| Variable | Dev Default | Production Comment |
|----------|-------------|-------------------|
| `RESEND_API_KEY` | | `# KEEP SECRET - Resend Dashboard > API Keys` |

---

### Email: AutoSend

Include when: AutoSend selected.

| Variable | Dev Default | Production Comment |
|----------|-------------|-------------------|
| `AUTOSEND_API_KEY` | | `# KEEP SECRET - AutoSend Dashboard > Settings > API Keys` |
| `FROM_EMAIL` | `noreply@localhost` | `# Your verified sending email (e.g., noreply@yourdomain.com)` |
| `FROM_NAME` | `My App` | `# Your app name shown in email From field` |

---

### AI: OpenAI

Include when: OpenAI or Vercel AI SDK with OpenAI selected.

| Variable | Dev Default | Production Comment |
|----------|-------------|-------------------|
| `OPENAI_API_KEY` | | `# KEEP SECRET - OpenAI Platform > API keys` |

---

### AI: Anthropic

Include when: Anthropic selected.

| Variable | Dev Default | Production Comment |
|----------|-------------|-------------------|
| `ANTHROPIC_API_KEY` | | `# KEEP SECRET - Anthropic Console > API keys` |

---

### AI: OpenRouter

Include when: OpenRouter selected.

| Variable | Dev Default | Production Comment |
|----------|-------------|-------------------|
| `OPENROUTER_API_KEY` | | `# KEEP SECRET - OpenRouter > Keys` |

---

### Storage: Cloudflare R2

Include when: Cloudflare R2 selected.

| Variable | Dev Default | Production Comment |
|----------|-------------|-------------------|
| `CLOUDFLARE_ACCOUNT_ID` | | `# Cloudflare Dashboard > R2 > Account ID (just the ID, not a URL)` |
| `R2_ACCESS_KEY_ID` | | `# Cloudflare Dashboard > R2 > Manage R2 API Tokens` |
| `R2_SECRET_ACCESS_KEY` | | `# KEEP SECRET - Cloudflare Dashboard > R2 > API Tokens` |
| `R2_BUCKET_NAME` | | `# Your R2 bucket name` |

---

### Deployment: Railway

Include when: Railway selected for deployment. These go in `.env.production` only.

| Variable | Dev Default | Production Comment |
|----------|-------------|-------------------|
| `RAILPACK_NO_SPA` | _(skip in .env)_ | `# Required for SSR apps on Railway - set to 1` |

---

## File Format Rules

### `.env` (local development)

```
# ---- Section Name ----
VAR_NAME=default_value
VAR_NAME=
```

- Section headers as comments: `# ---- Section Name ----`
- One blank line between sections
- Pre-fill localhost URLs where applicable
- Leave API keys and secrets as empty strings
- No quotes around values

### `.env.production` (production reference)

```
# ---- Section Name ----
# Where to find this value
# KEEP SECRET (if applicable)
VAR_NAME=
```

- Same section structure as `.env`
- Every variable has a comment above it explaining where to get the value
- Secret variables get an additional `# KEEP SECRET` line
- All values are empty - user fills them in and copies to deployment platform
- `NODE_ENV=production` is the only pre-filled value

### `.gitignore` rules

Always ensure these patterns exist in `.gitignore`:

```
.env
.env.local
.env.production
.env.*.local
```

If `.gitignore` exists, check if these patterns are already present. Only append missing ones.
If `.gitignore` does not exist, create it with these patterns plus common defaults (`node_modules/`, `.output/`, `dist/`, `.DS_Store`).

## Deduplication Rules

When multiple service groups include the same variable (e.g., `DATABASE_URL` appears in both database and auth groups):
- Include it only once, under the first service group that needs it
- The database group takes priority over auth for `DATABASE_URL`
- `APP_URL` in the App group and `BETTER_AUTH_URL` in the Auth group are separate variables (both needed)
