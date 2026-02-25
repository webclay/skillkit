# Changelog

**Last Updated:** 2026-02-25

This file tracks work completed across sessions to help maintain context.

---

## Session Log

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
