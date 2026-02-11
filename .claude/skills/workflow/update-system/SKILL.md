---
name: update-skillkit
description: Check for and apply updates to SkillKit. Trigger words - skillkit update, update skillkit, skillkit version, skillkit restore, skillkit rollback
---

# Update SkillKit

Handles version checking, updates, and rollback for the SkillKit system.

## When to Use This Skill

- User says "skillkit update" or "update skillkit"
- User asks about the SkillKit version
- User wants to restore a previous SkillKit version
- User mentions "skillkit rollback" or "skillkit restore"

**Note:** Do NOT use this for "update stack" (which updates project dependencies) or generic "update" commands.

## Version Checking

### How It Works

1. Read local version from `.claude/version.json`
2. Compare with latest version (user provides or checks GitHub)
3. Show what's new if update available

### Version File Format

`.claude/version.json`:
```json
{
  "version": "1.0.0",
  "releaseDate": "2025-01-04"
}
```

## Update Workflow

### When User Says "skillkit update"

1. **Check current version**
   - Read `.claude/version.json`
   - Show current version to user

2. **Ask update method**
   ```
   Current version: 1.0.0

   How would you like to update?

   A. Automatic - I'll guide you through downloading from GitHub
   B. Manual - You already have the new files, I'll help copy them

   Your choice?
   ```

3. **Guide through update** (see flows below)

### Automatic Update Flow

```
Step 1: Download the latest release from GitHub
→ Go to: [repository URL]/releases
→ Download the .claude folder ZIP

Step 2: Extract the ZIP

Step 3: I'll help you copy the files...

First, copy the `skills/` folder from the download
to your project's `.claude/skills/` folder.

Done? (yes/no)

[Continue with each folder/file...]
```

### Manual Update Flow

If user already downloaded:

```
Great! You have the new files ready.

Copy these folders (replace when asked):
1. skills/
2. templates/
3. commands/

Copy these files:
1. CLAUDE.md
2. version.json

Your project/projectbrief.md, project/tasks.md, and project/changelog.md are SAFE -
they won't be touched.

Done? (yes/no)
```

## File Categories

### Never Update (User Data)

These files are NEVER modified during updates:
- `.claude/project/projectbrief.md` - User's project configuration
- `.claude/project/tasks.md` - User's task tracking
- `.claude/project/changelog.md` - User's session history
- `.claude/project/dev-context.md` - User's architectural decisions and project config
- `.claude/project/.setup-complete` - User's setup marker
- Any custom files user added

### Always Update (System Files)

These files are updated:
- `.claude/CLAUDE.md` - Main instructions
- `.claude/version.json` - Version tracking
- `.claude/skills/**/*` - All skills
- `.claude/templates/**/*` - All templates
- `.claude/commands/**/*` - All slash commands

## Backup Strategy

### Before Updating

Create backup of current system files:

```
.claude/.backup-v1.0.0/
├── CLAUDE.md
├── skills/
├── templates/
└── commands/
```

### Backup Command

```
I'll create a backup before updating...

Created backup at: .claude/.backup-v1.0.0/

Your user files (projectbrief, tasks, changelog) are safe
and won't be included in the backup.

Ready to proceed with update? (yes/no)
```

## Rollback

### When User Says "restore" or "rollback"

1. Check for existing backups
2. Show available versions
3. Restore selected version

```
Found backup from before last update:
- .backup-v1.0.0 (created: 2025-01-04)

Restore to version 1.0.0? This will undo the update.

Your project/projectbrief.md and project/tasks.md won't be affected.

Restore? (yes/no)
```

### Restore Process

1. Copy backup files back to main locations
2. Update version.json to restored version
3. Verify files are restored
4. Confirm to user

## Version Comparison

### Semantic Versioning

Format: `MAJOR.MINOR.PATCH`
- **Major** (1.x.x) - Breaking changes, new architecture
- **Minor** (x.1.x) - New features, backward compatible
- **Patch** (x.x.1) - Bug fixes, small improvements

### Update Priority

- **Major** - Review changes carefully before updating
- **Minor** - Safe to update, new features available
- **Patch** - Recommended, fixes issues

## Commands Reference

| User Says | Action |
|-----------|--------|
| "skillkit update" | Check for updates and guide through installation |
| "skillkit version" | Show current SkillKit version |
| "skillkit restore" | Rollback to previous version from backup |
| "skillkit check" | Just check if update available (don't install) |

## Safety Notes

1. **User data is sacred** - Never touch projectbrief, tasks, or changelog
2. **Always backup first** - Create backup before any update
3. **Verify after update** - Check that skills load correctly
4. **Keep old backups** - Maintain last 2-3 backups for safety

## Example Conversations

### Checking for Updates

```
User: skillkit update

Claude: Checking your SkillKit version...

Current version: 1.1.0

Would you like to:
A. Check GitHub for the latest release
B. I already downloaded an update

Your choice?
```

### Restoring Previous Version

```
User: skillkit restore

Claude: Looking for backups...

Found backup:
- v1.0.0 (from 2 days ago)

This will restore:
- CLAUDE.md
- All skills
- All templates

Your project/projectbrief.md and project/tasks.md will NOT be changed.

Restore to v1.0.0? (yes/no)
```

## Troubleshooting

### Update Didn't Work

1. Check that all files were copied
2. Verify version.json shows new version
3. Try running `setup` to verify configuration

### Missing Skills After Update

1. Check `.claude/skills/` folder exists
2. Verify skill folders have SKILL.md files
3. Re-copy skills folder from update package

### Backup Not Found

If no backup exists:
1. Cannot rollback automatically
2. User needs to manually restore from GitHub
3. Create backup before future updates
