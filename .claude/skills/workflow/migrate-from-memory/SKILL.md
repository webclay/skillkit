---
name: migrate-from-memory
description: Migrate from old claude-memory system to new skills system. Trigger words - migrate, upgrade skills, convert memory, old system, claude-memory
---

# Migrate from Claude Memory

Migrates projects from the old claude-memory system to the new skills-based system.

## When to Use This Skill

- User says "migrate" or "upgrade claude memory"
- Project has `stacks/` directory (old system)
- Project has `cm/` directory (old commands)
- User copied new skills/ folder and wants to complete migration

## Prerequisites

Before running this migration, ensure:
1. The `skills/` folder has been copied into `.claude/`
2. The `templates/` folder has been copied (optional but recommended)

Note: Do NOT copy CLAUDE.md manually - this skill handles that.

## Instructions

### Step 1: Detect Old System

Check for old system markers:
- `.claude/stacks/` exists
- `.claude/cm/` exists
- `.claude/triggers.md` exists

If none found, inform user this is not an old claude-memory project.

### Step 2: Verify User Content

Confirm these files exist (will be preserved and moved to project/ folder):
- projectbrief.md (moves to project/projectbrief.md)
- tasks.md (moves to project/tasks.md)
- changelog.md (moves to project/changelog.md)

If any are missing, warn user before proceeding.

### Step 3: Extract Custom Content from Old CLAUDE.md

Read the old `.claude/CLAUDE.md` and look for:
- Any custom rules added by user (not standard cm system rules)
- Project-specific instructions
- Custom commands beyond the standard cm commands

Save any custom content to merge into new CLAUDE.md.

### Step 4: Prepare New CLAUDE.md

Read `CLAUDE-template.md` from this skill folder.

If custom content was found in Step 3:
- Add a "## Project-Specific Rules" section at the end
- Insert the custom content there

### Step 5: Remove Old System Files

Delete these directories and files (if they exist):
- `stacks/` (entire directory)
- `cm/` (entire directory)
- `commands/` (entire directory, if exists - duplicate of cm/)
- `guides/` (entire directory)
- `triggers.md`
- `mistakes.md`
- `standards.md`
- `00-start-here.md`
- `01-rules-definitions.md`
- `04-testing.md`
- `07-documentation-maintenance.md`
- `SETUP.md`
- `VERSION` (replaced by version.json)
- Old `CLAUDE.md`

### Step 6: Install New CLAUDE.md

Write the prepared CLAUDE.md (with any merged custom content) to `.claude/CLAUDE.md`.

### Step 7: Create New System Files

Create `project/.setup-complete`:
```
Setup completed: [Current Date]
Project: [Project name from project/projectbrief.md]
Migrated from: claude-memory
```

Create `version.json`:
```json
{
  "version": "1.0.0",
  "updated": "[Current Date]"
}
```

Create `settings.json` (if not exists):
```json
{
  "model": "opusplan"
}
```

### Step 8: Update project/projectbrief.md

Add "Relevant Skills" section if missing, mapping detected tech stack to new skill paths:

```markdown
## Relevant Skills

Based on your tech stack, reference these skills for implementation patterns:

| Technology | Skill Location |
|------------|----------------|
| Database (Prisma) | `.claude/skills/database/prisma/` |
| Auth (Better Auth) | `.claude/skills/auth/better-auth/` |
| UI (Shadcn) | `.claude/skills/ui/shadcn/` |
```

### Step 9: Report

Show summary to user:

```
Migration Complete!

Preserved (user content, now in project/ folder):
- project/projectbrief.md
- project/tasks.md
- project/changelog.md

Custom rules migrated: [yes/no]

Removed (old system files):
- stacks/ directory
- cm/ directory
- [X] other deprecated files

Created (new system files):
- CLAUDE.md (new version)
- project/.setup-complete
- version.json
- settings.json

Your project is now using the new skills system!
```

## Old vs New System Reference

| Old Location | New Location |
|--------------|--------------|
| `stacks/database/database-prisma.md` | `skills/database/prisma/SKILL.md` |
| `stacks/auth/auth-better-auth.md` | `skills/auth/better-auth/SKILL.md` |
| `cm/log.md` | `skills/workflow/session-log/SKILL.md` |
| `cm/setup.md` | `skills/workflow/project-setup/SKILL.md` |
| `cm/status.md` | `skills/workflow/task-tracker/SKILL.md` |
| `triggers.md` | Built into CLAUDE.md |
| `mistakes.md` | "Common Mistakes" section in CLAUDE.md |

## Verification

After migration:
- [ ] New `CLAUDE.md` in place (check for "# Project Instructions" header)
- [ ] `project/.setup-complete` exists
- [ ] `skills/` directory exists with all categories
- [ ] `stacks/` directory removed
- [ ] `cm/` directory removed
- [ ] `commands/` directory removed (if it existed)
- [ ] User content preserved in project/ folder (projectbrief, tasks, changelog)
- [ ] `project/projectbrief.md` has "Relevant Skills" section

## Rollback

If something goes wrong, the user can:
1. Restore from git (if committed before migration)
2. Re-copy old files from claude-memory repo
3. The skills/ folder doesn't affect old system files until migration runs
