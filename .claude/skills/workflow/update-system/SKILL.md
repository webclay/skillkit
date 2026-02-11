---
name: update-skillkit
description: Check for and apply updates to SkillKit. Trigger words - skillkit update, update skillkit, skillkit version, skillkit restore, skillkit rollback
---

# Update SkillKit

Auto-fetch updates from GitHub, with backup and rollback support.

## When to Use This Skill

- User says "skillkit update" or "update skillkit"
- User asks about the SkillKit version
- User wants to restore a previous SkillKit version
- User mentions "skillkit rollback" or "skillkit restore"

**Note:** Do NOT use this for "update stack" (which updates project dependencies) or generic "update" commands.

## Commands Reference

| User Says | Action |
|-----------|--------|
| "skillkit update" | Check for updates and auto-install if available |
| "skillkit version" | Show current SkillKit version |
| "skillkit restore" | Rollback to previous version from backup |
| "skillkit check" | Just check if update available (don't install) |

---

## "skillkit version"

Read `.claude/version.json` and display:

```
SkillKit v1.2.0 (released 2026-02-11)
Repository: grasman79/skillkit
```

---

## "skillkit check"

1. Read `.claude/version.json` to get `version` and `repository`
2. Fetch remote version:
   ```bash
   curl -s https://raw.githubusercontent.com/{repository}/main/.claude/version.json
   ```
3. Compare versions using semantic versioning
4. Report result:
   - If up to date: `SkillKit v1.2.0 is up to date.`
   - If update available: `SkillKit update available: v1.2.0 -> v1.3.0. Run "skillkit update" to install.`
   - If fetch fails: `Could not check for updates (network error). Current version: v1.2.0`

---

## "skillkit update"

### Step 1: Check Versions

1. Read `.claude/version.json` - extract `version` (local) and `repository`
2. Fetch the remote version:
   ```bash
   curl -s https://raw.githubusercontent.com/{repository}/main/.claude/version.json
   ```
3. Parse the remote JSON to get the remote `version`
4. Compare using semantic versioning (MAJOR.MINOR.PATCH)

If local version >= remote version, report:
```
SkillKit v1.2.0 is already up to date.
```
And stop.

### Step 2: Confirm with User

Show the user what will happen:
```
SkillKit update available: v1.2.0 -> v1.3.0

This will update:
- All skills (.claude/skills/)
- All templates (.claude/templates/)
- All commands (.claude/commands/)
- Main instructions (.claude/CLAUDE.md)
- Version file (.claude/version.json)

Your project files are SAFE and won't be touched:
- project/projectbrief.md
- project/tasks.md
- project/changelog.md
- project/dev-context.md

A backup of your current version will be created.

Proceed with update? (yes/no)
```

Wait for user confirmation before continuing.

### Step 3: Create Backup

```bash
mkdir -p .claude/.backup-v{LOCAL_VERSION}
cp -r .claude/skills .claude/.backup-v{LOCAL_VERSION}/skills
cp -r .claude/templates .claude/.backup-v{LOCAL_VERSION}/templates
cp -r .claude/commands .claude/.backup-v{LOCAL_VERSION}/commands
cp .claude/CLAUDE.md .claude/.backup-v{LOCAL_VERSION}/CLAUDE.md
cp .claude/version.json .claude/.backup-v{LOCAL_VERSION}/version.json
```

Confirm backup was created:
```
Backup created at .claude/.backup-v1.2.0/
```

### Step 4: Download and Extract

```bash
# Download the latest tarball from main branch
curl -sL "https://api.github.com/repos/{repository}/tarball/main" -o /tmp/skillkit-update.tar.gz

# Create extraction directory and extract
mkdir -p /tmp/skillkit-update
tar xzf /tmp/skillkit-update.tar.gz -C /tmp/skillkit-update --strip-components=1
```

Verify extraction succeeded by checking the extracted `.claude/version.json` exists:
```bash
cat /tmp/skillkit-update/.claude/version.json
```

If extraction fails, report the error and stop. The backup is still intact.

### Step 5: Apply Update

```bash
# Remove old system files
rm -rf .claude/skills .claude/templates .claude/commands

# Copy fresh system files
cp -r /tmp/skillkit-update/.claude/skills .claude/skills
cp -r /tmp/skillkit-update/.claude/templates .claude/templates
cp -r /tmp/skillkit-update/.claude/commands .claude/commands
cp /tmp/skillkit-update/.claude/CLAUDE.md .claude/CLAUDE.md
cp /tmp/skillkit-update/.claude/version.json .claude/version.json
```

### Step 6: Clean Up

```bash
rm -rf /tmp/skillkit-update /tmp/skillkit-update.tar.gz
```

### Step 7: Verify and Report

Read the new `.claude/version.json` and confirm it matches the remote version.

Report:
```
SkillKit updated successfully: v1.2.0 -> v1.3.0

Updated:
- Skills, templates, and commands
- Main instructions (CLAUDE.md)

Your project files were not touched.
Backup of v1.2.0 saved at .claude/.backup-v1.2.0/

To rollback: run "skillkit restore"
```

---

## "skillkit restore" / "skillkit rollback"

### Step 1: Find Backups

List backup directories:
```bash
ls -d .claude/.backup-v*/ 2>/dev/null
```

If no backups found:
```
No backups found. Cannot rollback.
You can manually restore from GitHub: https://github.com/{repository}
```

### Step 2: Show Available Backups

```
Found backup(s):
- v1.2.0 (at .claude/.backup-v1.2.0/)

Your project files (projectbrief, tasks, changelog) will NOT be affected.

Restore to v1.2.0? (yes/no)
```

If multiple backups exist, list all and ask which one to restore.

### Step 3: Restore

```bash
# Remove current system files
rm -rf .claude/skills .claude/templates .claude/commands

# Restore from backup
cp -r .claude/.backup-v{VERSION}/skills .claude/skills
cp -r .claude/.backup-v{VERSION}/templates .claude/templates
cp -r .claude/.backup-v{VERSION}/commands .claude/commands
cp .claude/.backup-v{VERSION}/CLAUDE.md .claude/CLAUDE.md
cp .claude/.backup-v{VERSION}/version.json .claude/version.json
```

### Step 4: Verify and Report

Read restored `.claude/version.json` and confirm version matches.

```
Restored SkillKit to v1.2.0.

Your project files were not affected.
The backup has been kept at .claude/.backup-v1.2.0/ in case you need it again.
```

---

## File Categories

### Never Update (User Data)

These files are NEVER modified during updates:
- `.claude/project/projectbrief.md` - User's project configuration
- `.claude/project/tasks.md` - User's task tracking
- `.claude/project/changelog.md` - User's session history
- `.claude/project/dev-context.md` - User's architectural decisions
- `.claude/project/.setup-complete` - User's setup marker
- `.claude/settings.json` - User's IDE settings
- `.claude/settings.local.json` - User's local settings
- `.claude/MAINTAINER_GUIDE.md` - SkillKit repo only (gitignored)
- Any custom files user added

### Always Update (System Files)

These files are replaced during updates:
- `.claude/CLAUDE.md` - Main instructions
- `.claude/version.json` - Version tracking
- `.claude/skills/**/*` - All skills
- `.claude/templates/**/*` - All templates
- `.claude/commands/**/*` - All slash commands

## Semantic Versioning

Format: `MAJOR.MINOR.PATCH`
- **Major** (1.x.x) - Breaking changes, new architecture
- **Minor** (x.1.x) - New features, multiple skills, backward compatible
- **Patch** (x.x.1) - Bug fixes, single skill, small improvements

## Safety Notes

1. **User data is sacred** - Never touch project/, settings, or custom files
2. **Always backup first** - Create backup before any update
3. **Verify after update** - Check that version.json shows new version
4. **Keep backups** - Don't delete backups automatically; let users manage them
5. **Ask before proceeding** - Always get user confirmation before applying updates

## Troubleshooting

### Network Error During Version Check

```
Could not reach GitHub. Check your internet connection.
Current local version: v1.2.0
```

Skip the update; don't block the user.

### Download Failed

If the tarball download fails:
1. Check internet connection
2. Try again: `curl -sL "https://api.github.com/repos/{repository}/tarball/main" -o /tmp/skillkit-update.tar.gz`
3. If GitHub API rate limited (403), wait a few minutes and retry
4. Backup is untouched - no damage done

### Extraction Failed

If tar extraction fails:
1. Delete the corrupted file: `rm /tmp/skillkit-update.tar.gz`
2. Re-download and try again
3. Backup is untouched

### Update Broke Something

Run `skillkit restore` to rollback to the backup.

### No Backup Found for Rollback

1. Cannot rollback automatically
2. Download the desired version manually from GitHub
3. Copy `.claude/skills/`, `.claude/templates/`, `.claude/commands/`, `.claude/CLAUDE.md`, and `.claude/version.json` from the download

### GitHub API Rate Limit

Unauthenticated requests: 60/hour. If rate limited:
- Wait a few minutes
- Or use `gh api repos/{repository}/tarball/main > /tmp/skillkit-update.tar.gz` (uses authenticated GitHub CLI)
