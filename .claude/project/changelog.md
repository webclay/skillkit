# Changelog

**Last Updated:** 2026-03-02

This file tracks work completed across sessions to help maintain context.

---

## Session Log

### 2026-03-02 - Content Website Checklist: Database URL Warning

**Summary:** Added critical Step 3 to the personal content website checklist emphasizing that the template's `DATABASE_URI` must be changed to the new project's Postgres database URL before running anything. The template ships with a connection string pointing to the template's own database.

**Completed:**
- Added **Step 3: Update the database connection URL** (marked CRITICAL) to `content-website-checklist.md`
- Added double-check reminder in Step 4 (SkillKit setup) since setup may overwrite `.env`
- Renumbered all subsequent steps (now 18 steps total, up from 17)
- Updated MEMORY.md reference with new step count and key gotcha note

**Files Changed (auto-memory only, no repo changes):**
- `~/.claude/projects/.../memory/content-website-checklist.md`
- `~/.claude/projects/.../memory/MEMORY.md`

---

### 2026-03-02 - Payload Auto-Seeding Admin Users

**Summary:** Added auto-seeding admin user pattern to Payload CMS skill, headless-cms reference, and content website setup questionnaire. User-specific email config stored in auto-memory, resolved per-project into `.env` and `dev-context.md`.

**Completed:**
- Added `onInit` hook pattern to `.claude/skills/cms/payload/headless-cms.md` (Standard payload.config.ts section)
- Added Auto-Seeding Admin Users section to `.claude/skills/cms/payload/SKILL.md`
- Updated `.claude/skills/workflow/project-setup/content-questionnaire.md`:
  - Added Admin Seeding Variables section (reads from MEMORY.md, resolves `{project}` placeholder)
  - Added `PAYLOAD_ADMIN_EMAILS` and `PAYLOAD_ADMIN_PASSWORD` to all three .env templates (PostgreSQL, SQLite/D1, MongoDB)
  - Updated Step 4 from "Create Admin Account" to "Log Into Admin Panel" with conditional seeding/manual flow
  - Updated initial tasks checklist
- Bumped version to 1.2.14

**Key Decisions:**
- Skill files stay generic (no user-specific emails) - safe for public GitHub
- User-specific seeding config (email templates, password) stored in auto-memory (`## Payload Admin Seeding`)
- Setup wizard reads memory, resolves `{project}` to lowercase folder name, writes to `.env` and `dev-context.md`
- Uses `PAYLOAD_ADMIN_EMAILS` (comma-separated) instead of single `PAYLOAD_PROJECT_NAME` - more flexible
- If no memory config exists, wizard skips seeding entirely (manual registration fallback)

**Files Changed:**
- `.claude/skills/cms/payload/SKILL.md` (added Auto-Seeding section)
- `.claude/skills/cms/payload/headless-cms.md` (added Standard payload.config.ts section)
- `.claude/skills/workflow/project-setup/content-questionnaire.md` (admin seeding flow, env templates, getting started guide)
- `README.md` (version bump, changelog entry)
- `.claude/version.json` (bumped to 1.2.14)

---

### 2026-03-02 - Railway Payload Deployment Guide

**Summary:** Added a step-by-step Railway deployment guide for content websites (Astro + Payload CMS monorepo). Fixed incorrect `DATABASE_URL` variable name to `DATABASE_URI` in the content website setup questionnaire.

**Completed:**
- Created `.claude/skills/workflow/project-setup/railway-payload-deploy.md` - Complete Railway deployment flow (project creation, Postgres, GitHub connection, monorepo paths, 3 required environment variables, domain setup, troubleshooting)
- Fixed `DATABASE_URL` to `DATABASE_URI` in content-questionnaire.md (Payload CMS uses `DATABASE_URI`, not `DATABASE_URL`)
- Added Railway deployment reference link to content-questionnaire.md (triggers when user selects Railway)
- Added Railway section to `cms/payload/deployment.md` with the 3 required variables and link to full guide

**Key Decisions:**
- Railway Payload deployment requires exactly 3 env vars: `DATABASE_URI`, `PAYLOAD_PUBLIC_SERVER_URL`, `PAYLOAD_SECRET`
- Used Railway template syntax for dynamic values: `${{Postgres.DATABASE_URL}}` and `https://${{RAILWAY_PUBLIC_DOMAIN}}`
- Reference file lives in project-setup (not Payload skill) since it's part of the setup wizard flow
- Payload deployment.md links to the reference file rather than duplicating content

**Files Changed:**
- `.claude/skills/workflow/project-setup/railway-payload-deploy.md` (new)
- `.claude/skills/workflow/project-setup/content-questionnaire.md` (fixed DATABASE_URI, added Railway reference)
- `.claude/skills/cms/payload/deployment.md` (added Railway section)

---

### 2026-02-27 - Agent Skills Spec Compliance Audit

**Summary:** Audited all 64 SkillKit skills against the official Anthropic Agent Skills specification (agentskills.io/specification) and the official Anthropic skills repo (github.com/anthropics/skills). Fixed all compliance issues: name/directory mismatches, duplicate skill names, oversized files, and description quality.

**Completed:**
- Fixed `update-system` name/directory mismatch (frontmatter said `update-skillkit`, directory was `update-system`)
- Renamed `tooling/ultracite` to `tooling/ultracite-setup` to resolve duplicate `ultracite` name conflict with `workflow/ultracite`
- Split 5 oversized SKILL.md files into main files + reference files:
  - Payload CMS (2091 to 297 lines) + 7 reference files
  - Supabase (964 to 191 lines) + 5 reference files
  - Expo (802 to 295 lines) + 2 reference files
  - Astro (577 to 415 lines) + 1 reference file
  - TanStack Start (518 to 341 lines) + 1 reference file
- Updated all 64 SKILL.md description fields from old "Trigger words -" pattern to new "Use this skill when... Activate when the user mentions..." pattern per Anthropic spec
- Updated README.md Tooling section for ultracite-setup rename
- Bumped version to 1.2.12

**Key Decisions:**
- Followed Anthropic's progressive disclosure model: metadata (~100 tokens), instructions (<500 lines), resources (on demand)
- Reference files kept one level deep from SKILL.md, no nested chains
- Descriptions now follow Anthropic's recommended format for better skill activation

**Files Changed:**
- 64 SKILL.md files (description field updates)
- `.claude/skills/workflow/update-system/SKILL.md` (name field fix)
- `.claude/skills/tooling/ultracite-setup/SKILL.md` (renamed from tooling/ultracite)
- `.claude/skills/cms/payload/SKILL.md` + 7 new reference files
- `.claude/skills/database/supabase/SKILL.md` + 5 new reference files
- `.claude/skills/platform/expo/SKILL.md` + 2 new reference files
- `.claude/skills/framework/astro/SKILL.md` + 1 new reference file
- `.claude/skills/framework/tanstack-start/SKILL.md` + 1 new reference file
- `README.md` (version, changelog, tooling section)
- `.claude/version.json` (bumped to 1.2.12)

---

### 2026-02-27 - Official Railway Skill Integration

**Summary:** Replaced the TanStack Start-specific Railway deployment skill with the official comprehensive skill from `railwayapp/railway-skills`. The new skill covers the full Railway platform with a routing table pointing to 5 reference files.

**Completed:**
- Replaced `.claude/skills/deployment/railway/SKILL.md` with official Railway skill (resource model, preflight checks, routing table, setup decision flow, composition patterns)
- Created 5 reference files in `.claude/skills/deployment/railway/references/`:
  - `setup.md` - Projects, services, databases, templates, workspaces
  - `deploy.md` - Deploy, redeploy, restart, build config, monorepo patterns, Railpack
  - `configure.md` - Environments, variables, config patches, domains, networking, multi-region
  - `operate.md` - Health checks, logs (filtered/time-bounded), metrics via GraphQL, failure triage
  - `request.md` - Railway GraphQL API, Central Station community search, official docs endpoints
- Created `.claude/skills/deployment/railway/scripts/railway-api.sh` - GraphQL API helper script
- Updated `.claude/CLAUDE.md` routing table entry from TanStack-specific to generic Railway
- Updated `README.md` skill description and added Railway entry to v1.2.11 changelog

**Key Decisions:**
- Replaced rather than kept alongside the old skill - the old one was project-specific (TanStack Start + Nitro SSR), which violates SkillKit's principle that skills should be generic
- Adapted frontmatter to SkillKit format (removed `allowed-tools` from Railway's plugin system, kept our `name`/`description` with trigger words)
- No version bump needed - added to existing v1.2.11 (same session as Resend integration)

**Files Changed:**
- `.claude/skills/deployment/railway/SKILL.md` (rewritten)
- `.claude/skills/deployment/railway/references/setup.md` (new)
- `.claude/skills/deployment/railway/references/deploy.md` (new)
- `.claude/skills/deployment/railway/references/configure.md` (new)
- `.claude/skills/deployment/railway/references/operate.md` (new)
- `.claude/skills/deployment/railway/references/request.md` (new)
- `.claude/skills/deployment/railway/scripts/railway-api.sh` (new)
- `.claude/CLAUDE.md` (updated routing table)
- `README.md` (updated skill description, added changelog entry)

---

### 2026-02-27 - Official Resend Skills Integration

**Summary:** Integrated all three official Resend agent skills (resend-skills, email-best-practices, react-email) into SkillKit's existing Resend skill, following the same pattern used for BetterAuth (official- prefix, SKILL.md as router/index).

**Completed:**
- Downloaded and saved 20 official reference files from three Resend GitHub repos:
  - resend/resend-skills: send-email, templates, inbound, agent-inbox
  - resend/email-best-practices: deliverability, compliance, transactional-emails, sending-reliability, webhooks-events, list-management, marketing-emails, email-capture, email-types, transactional-catalog
  - resend/react-email: react-email overview, react-components, react-patterns, react-i18n, react-sending
- Rewrote `.claude/skills/email/resend/SKILL.md` as a comprehensive router/index with guide table mapping 21 topics to their files
- Updated all cross-references in official files (relative paths to `official-*.md` pattern, external refs to GitHub URLs)
- Added 4 new routing entries to `.claude/CLAUDE.md` (deliverability, compliance, receiving, React Email)
- Updated README.md skill description and added v1.2.11 changelog entry
- Bumped version to 1.2.11

**Key Decisions:**
- Followed BetterAuth pattern: `official-` prefix for downloaded files, SKILL.md as router
- Cross-references within files updated to match flat file structure (no subdirectories)
- References to files we didn't download (send-email sub-references) point to GitHub URLs
- Agent inbox skill condensed with link to full GitHub version for complete implementations
- Best practices skill serves as router to 10 topic-specific resource files

**Files Changed:**
- `.claude/skills/email/resend/SKILL.md` (rewritten as router)
- `.claude/skills/email/resend/official-send-email.md` (new)
- `.claude/skills/email/resend/official-templates.md` (new)
- `.claude/skills/email/resend/official-inbound.md` (new)
- `.claude/skills/email/resend/official-agent-inbox.md` (new)
- `.claude/skills/email/resend/official-best-practices.md` (new)
- `.claude/skills/email/resend/official-react-email.md` (new)
- `.claude/skills/email/resend/official-deliverability.md` (new)
- `.claude/skills/email/resend/official-compliance.md` (new)
- `.claude/skills/email/resend/official-transactional-emails.md` (new)
- `.claude/skills/email/resend/official-sending-reliability.md` (new)
- `.claude/skills/email/resend/official-webhooks-events.md` (new)
- `.claude/skills/email/resend/official-list-management.md` (new)
- `.claude/skills/email/resend/official-marketing-emails.md` (new)
- `.claude/skills/email/resend/official-email-capture.md` (new)
- `.claude/skills/email/resend/official-email-types.md` (new)
- `.claude/skills/email/resend/official-transactional-catalog.md` (new)
- `.claude/skills/email/resend/official-react-components.md` (new)
- `.claude/skills/email/resend/official-react-patterns.md` (new)
- `.claude/skills/email/resend/official-react-i18n.md` (new)
- `.claude/skills/email/resend/official-react-sending.md` (new)
- `.claude/CLAUDE.md` (added 4 routing entries)
- `README.md` (updated skill description, version, changelog)
- `.claude/version.json` (bumped to 1.2.11)

---

### 2026-02-25 - Official Ultracite Skill Integration

**Summary:** Rewrote the Ultracite linting skill based on the official `haydenbleasel/ultracite` skill from skills.sh, significantly expanding its coverage.

**Completed:**
- Rewrote `.claude/skills/workflow/ultracite/SKILL.md` with official skill content
- Added support for three linter backends (Biome, ESLint + Prettier, Oxlint + Oxfmt)
- Added linter detection logic (biome.jsonc, eslint.config.mjs, .oxlintrc.json)
- Added full CLI commands (check, fix, doctor, init)
- Added non-interactive init flags for CI/scripted setups
- Added framework presets (React, Next, Solid, Vue, Svelte, Qwik, Remix, Angular, Astro, NestJS)
- Added comprehensive code standards (formatting, style, type safety, async, React, performance, security)
- Added troubleshooting section with `doctor` command
- Added context-aware rule overrides (config files, tests, scripts, storybook, build output)
- Updated README.md changelog and version to 1.2.10
- Updated version.json to 1.2.10

**Key Decisions:**
- Kept SkillKit-specific patterns (`[runner]` placeholder, pre-commit workflow, output format)
- Content sourced from skills.sh page and raw GitHub code-standards.md reference

**Files Changed:**
- `.claude/skills/workflow/ultracite/SKILL.md`
- `README.md`
- `.claude/version.json`

---

### 2026-02-13 - Payload CMS Skill, Astro Enhancements, and Loading Best Practices

**Summary:** Created comprehensive Payload CMS skill, enhanced Astro skill with headless CMS integration and View Transitions, added server-first rendering best practices, and documented loader caching patterns.

**Completed:**
- Created new skill: `.claude/skills/cms/payload/SKILL.md` - Full Payload CMS reference covering collections, field types, blocks, media, rich text, access control, hooks, REST API, Local API, Form Builder plugin, monorepo deployment (Railway + Cloudflare Pages), and Bun runtime compatibility
- Updated `.claude/skills/framework/astro/SKILL.md` - Added headless CMS integration section (Astro + Payload), View Transitions for SPA-like navigation, prefetching strategies
- Updated `.claude/skills/framework/tanstack-start/SKILL.md` - Added "Preventing UI Flash / Double-Load" section with BAD/GOOD patterns, loader caching with staleTime, TanStack Query ensureQueryData, and Link preloading
- Updated `.claude/skills/workflow/best-practices/SKILL.md` - Added "Server-First Rendering" section covering the double-load flash problem, correct skeleton patterns, caching guidance
- Updated `.claude/CLAUDE.md` - Added Payload CMS and Astro to skills routing table
- Cross-referenced Payload and Astro skills so they link to each other

**Key Decisions:**
- Skeletons during navigation are good (pendingComponent/Suspense) - the problem is skeletons that replace already-rendered content (useEffect pattern)
- Loader caching (staleTime) ensures skeleton only shows on first visit, not on return navigation
- Payload recommended as headless CMS with Astro static frontend for content sites
- Monorepo structure with separate package.json files for Payload and Astro, deployed to different hosts
- Bun works as package manager for Payload but not as runtime - use Node.js runtime with --disable-transpile workaround if needed

**Files Changed:**
- `.claude/skills/cms/payload/SKILL.md` (new)
- `.claude/skills/framework/astro/SKILL.md` (updated)
- `.claude/skills/framework/tanstack-start/SKILL.md` (updated)
- `.claude/skills/workflow/best-practices/SKILL.md` (updated)
- `.claude/CLAUDE.md` (updated)

**Next Steps:**
- Build first Astro + Payload client website using the new skills
- Consider adding Payload Live Preview integration patterns

---

### 2026-02-11 - Auto-Fetch Update System

**Summary:** Upgraded the `skillkit update` command from a manual guided download process to a fully automated GitHub fetch. Also fixed the startup version check URL that was returning 404.

**Completed:**
- Rewrote `update-system/SKILL.md` with auto-fetch workflow (backup, download tarball, extract, replace system files, verify)
- Version checking now uses `raw.githubusercontent.com` instead of GitHub Releases API (no releases exist yet)
- Tarball download uses `api.github.com/repos/{repo}/tarball/main`
- Fixed CLAUDE.md startup version check to use same working URL
- Added `gh` CLI fallback for GitHub API rate limits
- Kept rollback support (restore from `.claude/.backup-v{version}/`)

**Key Decisions:**
- Use `raw.githubusercontent.com` for version checks (works without GitHub releases/tags)
- Use main branch tarball for downloads (works today, scales to tagged releases later)
- Always back up before updating, never auto-delete backups
- Protected files list expanded: settings.json, settings.local.json, MAINTAINER_GUIDE.md

**Files Changed:**
- `.claude/skills/workflow/update-system/SKILL.md` (rewritten)
- `.claude/CLAUDE.md` (fixed version check URL)

---

### 2026-02-11 - Workflow Simplification and Architecture Refactor

**Summary:** Major refactor based on Claude Code Insights report analysis across 135 sessions. Simplified git workflow from 4 commands to 3, moved project-specific config from CLAUDE.md to dev-context.md, added optional Greptile code review automation, and updated GitHub repo URL.

**Completed:**
- Created `/wrap-up` skill combining log + lint + build + commit + push + PR + merge + cleanup
- Removed `/push` skill and command (absorbed into /wrap-up)
- Removed `/merge` skill and command (absorbed into /wrap-up)
- Created `/wrap-up` command file
- Added optional Greptile code review cycle to wrap-up (polls for review, auto-fixes, merges)
- Moved Package Manager config from CLAUDE.md to project/dev-context.md
- Moved Code Review config from CLAUDE.md to project/dev-context.md
- Updated project-setup to write PM and Code Review to dev-context.md
- Updated questionnaire with Greptile recommendation (plain language for non-coders)
- Updated ultracite and wrap-up skills to read PM from dev-context.md
- Fixed dev-context.md path in update-system skill
- Fixed log command paths to use project/ prefix
- Updated all repo URLs from webclay/skillkit to grasman79/skillkit
- Updated git remote to new repo URL
- Made Co-Authored-By tag model-agnostic
- Added UI/styling scope rule to prevent over-application
- Updated create-branch to reference /wrap-up

**Key Decisions:**
- Skills must stay generic - project-specific config goes in project/dev-context.md
- CLAUDE.md is a pure template that can be safely overwritten during SkillKit updates
- dev-context.md is protected during updates (user data)
- Greptile integration is optional - configured per-project during setup
- Auto-merge (`gh pr merge --auto --squash --delete-branch`) for non-Greptile users

**Files Changed:**
- `.claude/skills/workflow/wrap-up/SKILL.md` (new)
- `.claude/commands/wrap-up.md` (new)
- `.claude/skills/workflow/push-branch/` (deleted)
- `.claude/skills/workflow/merge-branch/` (deleted)
- `.claude/commands/push.md` (deleted)
- `.claude/commands/merge.md` (deleted)
- `.claude/skills/workflow/project-setup/SKILL.md`
- `.claude/skills/workflow/project-setup/questionnaire.md`
- `.claude/skills/workflow/ultracite/SKILL.md`
- `.claude/skills/workflow/wrap-up/SKILL.md`
- `.claude/skills/workflow/create-branch/SKILL.md`
- `.claude/skills/workflow/update-system/SKILL.md`
- `.claude/commands/log.md`
- `.claude/CLAUDE.md`
- `.claude/version.json`
- `.claude/MAINTAINER_GUIDE.md`
- `.claude/project/changelog.md`

**Next Steps:**
- Bump version to 1.2.0 and create GitHub release
- Copy updated .claude folder to other projects using SkillKit

---

### 2026-02-09 - TanStack Start Security and Database Integration

**Summary:** Created comprehensive security integration for TanStack Start projects with Nitro middleware and updated database skills to prevent client bundle leaks.

**Completed:**
- Created new security skill: `.claude/skills/security/tanstack-nitro-security/SKILL.md`
  - Implements 6 critical HTTP security headers via Nitro middleware
  - X-Content-Type-Options, X-Frame-Options, X-XSS-Protection
  - Referrer-Policy, Permissions-Policy, Strict-Transport-Security
- Updated `.claude/skills/framework/tanstack-start/SKILL.md`
  - Added mandatory "Security Requirements" section
  - References tanstack-nitro-security skill
- Updated `.claude/skills/database/prisma/SKILL.md`
  - Added "Framework Integration" section with TanStack Start API route patterns
  - Shows correct vs incorrect usage to prevent bundle leaks
  - Includes Next.js patterns for comparison
- Updated `.claude/skills/database/drizzle/SKILL.md`
  - Added "Framework Integration" section with TanStack Start API route patterns
  - Shows correct vs incorrect usage to prevent bundle leaks
  - Includes Next.js patterns for comparison
- Updated `.claude/CLAUDE.md`
  - Added tanstack-nitro-security to skills reference table

**Key Decisions:**
- Recommend Drizzle as default ORM for TanStack Start (edge-ready, faster, TypeScript-native)
- Database queries MUST go in API routes (`app/routes/api/*`) not loaders to prevent bundle leaks
- Security headers are mandatory for all TanStack Start projects
- Skills are now interconnected - framework skill references security and database patterns

**Files Changed:**
- `.claude/skills/security/tanstack-nitro-security/SKILL.md` (new)
- `.claude/skills/framework/tanstack-start/SKILL.md`
- `.claude/skills/database/prisma/SKILL.md`
- `.claude/skills/database/drizzle/SKILL.md`
- `.claude/CLAUDE.md`

**Pull Request:**
- Created PR #1: https://github.com/grasman79/skillkit/pull/1
- Branch: feature/tanstack-start-security-integration
- Status: Ready for review

**Notes for Future Sessions:**
- All TanStack Start projects now automatically get security headers setup
- Database skills explicitly prevent the common mistake of importing db in loaders
- The security implementation follows OWASP best practices and user's production guide
- Skills form a cohesive system optimized for TanStack Start stack

---

### 2025-12-31 - Added Electric SQL Stack

**Summary:** Created a comprehensive Electric SQL stack guide. Electric SQL is a Postgres sync engine for building local-first applications with real-time data synchronization.

**Completed:**
- Created new stack file: `.claude/stacks/database/database-electric-sql.md`
- Documented core concepts: Shapes, ShapeStream, Shape classes
- Added TypeScript client API with all configuration options
- Included React integration with useShape hook
- Documented API proxy pattern for production security
- Added custom parsing, error handling, and deployment sections
- Integrated TanStack DB documentation with collections, live queries, and transactional mutations
- Referenced key blog post: "Local-first sync with Electric and TanStack DB"

**Files Changed:**
- `.claude/stacks/database/database-electric-sql.md` (new)

**Notes for Future Sessions:**
- Electric SQL works well with TanStack DB for local-first React apps
- The API proxy pattern is recommended for production deployments
- Shapes are the core primitive for selective data syncing

---
