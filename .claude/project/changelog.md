# Changelog

**Last Updated:** 2026-02-09

This file tracks work completed across sessions to help maintain context.

---

## Session Log

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
- Created PR #1: https://github.com/webclay/skillkit/pull/1
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
