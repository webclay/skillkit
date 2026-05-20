---
name: cloudflare-local-dev-tunnels
description: Expose local dev servers over Cloudflare Tunnel for sharing previews, testing webhooks, and accessing apps from other devices. Covers quick tunnels (random trycloudflare.com URLs), named tunnels (stable hostnames), Wrangler flags (--tunnel, --tunnel-name), Cloudflare Vite plugin tunnel config, security considerations, and Cloudflare Access protection. Trigger words - local dev tunnel, expose localhost, share preview, test webhook, trycloudflare, cloudflare tunnel dev, wrangler tunnel, vite tunnel, tunnel dev server, share local dev, public url local, ngrok alternative cloudflare
---

# Cloudflare Local Dev Tunnels

Expose your local dev server over a Cloudflare Tunnel so you can share a preview, test a webhook, or access your app from another device.

## Source of Truth

Official documentation: https://developers.cloudflare.com/workers/development-testing/local-dev-tunnels/

For full Cloudflare Tunnel/Zero Trust documentation: https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/

Always check the official docs when the skill content conflicts or when you need the latest details.

## When to Use This Skill

- Sharing a local dev preview with teammates or clients
- Testing inbound webhooks (Stripe, GitHub, payment providers, etc.) against localhost
- Accessing your dev server from a mobile device or different machine
- Quick demos without deploying to production
- Testing OAuth callbacks that require HTTPS endpoints

## When NOT to Use This Skill

- Deploying to production (use [workers-core](../workers-core/SKILL.md))
- Setting up a permanent tunnel for a hosted service (use full Cloudflare Tunnel docs)
- Configuring wrangler, bindings, or secrets (use [workers-core](../workers-core/SKILL.md))
- Framework-specific Workers setup (use `cloudflare/tanstack-start`, `cloudflare/astro`, etc.)

## Prerequisites

Read [workers-core](../workers-core/SKILL.md) for wrangler CLI fundamentals.

For named tunnels only: install `cloudflared` CLI and authenticate (see Named Tunnels section).

## Tunnel Types

| Type | Hostname | Persistence | Setup |
|------|----------|-------------|-------|
| Quick tunnel | Random `*.trycloudflare.com` | Session only - new URL each time | Zero config |
| Named tunnel | Stable custom hostname | Persists across sessions | Requires `cloudflared` auth + DNS |

## Quick Tunnels

Quick tunnels generate a random `*.trycloudflare.com` URL. No account or configuration needed - just start one and share the URL.

### Wrangler

**Interactive (recommended):**

```bash
# Start dev server normally
npx wrangler dev

# Press [t] to start/close the tunnel
# Wrangler prints the public URL
```

**Auto-start:**

```bash
# Tunnel opens automatically when dev server starts
npx wrangler dev --tunnel
```

### Cloudflare Vite Plugin

**Interactive:**

```bash
# Start Vite dev server
npx vite dev

# Press t + Enter to start/close the tunnel
```

**Auto-start in vite.config.ts:**

```typescript
import { defineConfig } from "vite";
import { cloudflare } from "@cloudflare/vite-plugin";

export default defineConfig({
  plugins: [
    cloudflare({
      tunnel: { autoStart: true },
    }),
  ],
});
```

## Named Tunnels

Named tunnels provide stable hostnames that persist across sessions. Useful when webhooks or external services need a fixed URL.

### Wrangler

```bash
npx wrangler dev --tunnel-name=my-tunnel
```

### Cloudflare Vite Plugin

```typescript
import { defineConfig } from "vite";
import { cloudflare } from "@cloudflare/vite-plugin";

export default defineConfig({
  plugins: [
    cloudflare({
      tunnel: { name: "my-tunnel" },
    }),
  ],
});
```

Auto-start a named tunnel:

```typescript
cloudflare({
  tunnel: { name: "my-tunnel", autoStart: true },
})
```

### Named Tunnel Prerequisites

Named tunnels require the `cloudflared` CLI and a Cloudflare account with a domain.

**1. Install cloudflared:**

```bash
# macOS
brew install cloudflared

# Debian/Ubuntu
sudo apt-get install cloudflared

# Arch
pacman -Syu cloudflared
```

**2. Authenticate:**

```bash
cloudflared tunnel login
# Opens browser - log in and select your domain
# Generates cert.pem in ~/.cloudflared/
```

**3. Create the tunnel:**

```bash
cloudflared tunnel create my-tunnel
# Outputs tunnel UUID and credentials file path
```

**4. Route DNS:**

```bash
cloudflared tunnel route dns my-tunnel preview.my-domain.com
# Creates CNAME record pointing to the tunnel
```

After this one-time setup, `wrangler dev --tunnel-name=my-tunnel` or the Vite plugin `tunnel.name` config will use the stable hostname.

## Vite Preview Host Validation

When using `vite preview` (not `vite dev`), you must whitelist the tunnel hostname in `preview.allowedHosts` or Vite will reject requests.

```typescript
import { defineConfig } from "vite";
import { cloudflare } from "@cloudflare/vite-plugin";

export default defineConfig({
  preview: {
    allowedHosts: [
      ".trycloudflare.com",   // Quick tunnels
      ".my-domain.com",        // Named tunnel domain
    ],
  },
  plugins: [
    cloudflare({
      tunnel: { name: "my-tunnel" },
    }),
  ],
});
```

This is not needed for `vite dev` - only `vite preview`.

## Security Considerations

Anyone with the tunnel URL can reach your dev server. Review what your app exposes before enabling a tunnel.

### What to check before opening a tunnel

- **Admin/debug endpoints** - ensure they require authentication
- **Remote bindings** - if your dev server connects to real KV, D1, R2, or other production resources, those are now accessible through the tunnel
- **Proxy routes** - code that proxies requests to private/internal services exposes those services
- **Source code with HMR** - `vite dev` serves source files for hot module replacement. Anyone on the tunnel can see your source code, file paths, and project structure

### Mitigation

- Use `vite preview` instead of `vite dev` when sharing publicly (serves built output only, no source code)
- Use a named tunnel protected by [Cloudflare Access](https://developers.cloudflare.com/cloudflare-one/policies/access/) for stable environments that need authentication
- Local dev routes like `/cdn-cgi/*` are automatically restricted and not exposed over the tunnel

## CLI Reference

| Command / Flag | Tool | What It Does |
|---------------|------|--------------|
| Press `[t]` during `wrangler dev` | Wrangler | Toggle quick tunnel on/off |
| `--tunnel` | Wrangler | Auto-start quick tunnel with dev server |
| `--tunnel-name=<name>` | Wrangler | Start named tunnel with dev server |
| Press `t + Enter` during `vite dev` | Vite plugin | Toggle quick tunnel on/off |
| `tunnel: { autoStart: true }` | Vite config | Auto-start quick tunnel |
| `tunnel: { name: "<name>" }` | Vite config | Use named tunnel |
| `cloudflared tunnel login` | cloudflared | Authenticate with Cloudflare account |
| `cloudflared tunnel create <name>` | cloudflared | Create a named tunnel |
| `cloudflared tunnel route dns <name> <hostname>` | cloudflared | Point DNS to tunnel |
| `cloudflared tunnel list` | cloudflared | List all tunnels on account |
| `cloudflared tunnel info <name>` | cloudflared | Show tunnel details |

## Common Gotchas

1. **Quick tunnel URL changes every session.** Don't hardcode it in webhook configs or external services. Use named tunnels for stable URLs.

2. **Named tunnels require one-time `cloudflared` setup.** You need `cloudflared tunnel login`, `create`, and `route dns` before `--tunnel-name` will work. This is a one-time setup per tunnel.

3. **`vite dev` exposes source code.** HMR and module serving reveal source files, paths, and project structure. Use `vite preview` for public-facing shares.

4. **`vite preview` rejects tunnel requests by default.** Add `.trycloudflare.com` or your domain to `preview.allowedHosts` in Vite config. This is not needed for `vite dev`.

5. **Tunnel is public by default.** Anyone with the URL can access your dev server. For sensitive work, protect named tunnels with Cloudflare Access.

6. **Remote bindings are live.** If your dev server uses `--remote` or connects to real KV/D1/R2, changes made through the tunnel affect real data.

7. **`/cdn-cgi/*` routes stay restricted.** Local dev infrastructure routes are not exposed through the tunnel - this is expected behavior, not a bug.

8. **Press `[t]` not `t + Enter` in Wrangler.** Wrangler and the Vite plugin use different key bindings. Wrangler: press `[t]`. Vite: press `t` then `Enter`.

9. **Named tunnel DNS propagation.** After running `cloudflared tunnel route dns`, the CNAME may take a few minutes to propagate. The tunnel itself is immediate.

10. **Port conflicts.** The tunnel connects to whatever port your dev server is running on. If you change the dev server port, the tunnel follows automatically - no reconfiguration needed.
