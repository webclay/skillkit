---
name: skill-creator
description: Creates new reusable skills from successful work patterns. Trigger words - create skill, new skill, remember this, save approach, make reusable, add skill
---

# Skill Creator

Packages a successful approach into a reusable skill that Claude can apply automatically in the future. This lets you capture what worked so you never have to explain it again.

## When to Use This Skill

- User says "save this as a skill" or "remember how we did this"
- A complex task was completed successfully and should be repeatable
- User wants to reuse a pattern across multiple projects
- User asks to capture or document a workflow they like

## Instructions

### Step 1: Analyze the Pattern

Review what was accomplished in the conversation:
1. Identify the task - What did we just do?
2. Extract the approach - What steps did we follow?
3. Note the decisions - What choices were made and why?
4. Find the triggers - When should this skill activate?

Ask the user:
- "What would you call this skill?"
- "When should I automatically use this approach?"
- "Are there variations we should handle differently?"

### Step 2: Structure the Skill

Organize the knowledge into files:

**SKILL.md** (required):
- Clear one-line description for auto-triggering
- "When to Use This Skill" section with bullet points
- "Instructions" section with step-by-step process
- "Examples" section showing expected outcomes

**Reference files** (optional, for detailed content):
- patterns.md - Code patterns and configurations
- examples.md - Extended usage examples
- troubleshooting.md - Common issues and fixes

### Step 3: Create the Files

Create the skill in `.claude/skills/[category]/[skill-name]/`:

```
[skill-name]/
├── SKILL.md           # Main instructions (under 200 lines)
├── patterns.md        # Code patterns (if needed)
└── examples.md        # Usage examples (if needed)
```

**Categories:**
- `workflow/` - Project management, session tracking
- `auth/` - Authentication systems
- `database/` - Database and ORM patterns
- `ui/` - UI libraries and components
- `payments/` - Payment processing
- `ai/` - AI SDK integration
- `platforms/` - Mobile, extensions, plugins

### Step 4: Verify

1. Read back the skill summary to the user
2. Confirm the trigger description makes sense
3. Ensure instructions are clear for non-coders

## Examples

**User:** "We just set up a great authentication flow. Save this as a skill."

**Response:**
1. Analyze what we built (Better Auth, email/password, OAuth)
2. Ask: "What should trigger this? 'Add login' or 'user authentication'?"
3. Create `skills/auth/custom-auth/SKILL.md`
4. Add reference files for configuration details
5. Confirm: "Skill created! Now whenever you mention user login, I'll follow this approach."

---

**Skill Template:**

```markdown
---
name: skill-name
description: Clear description of what this skill does. Use when [trigger conditions].
---

# Skill Name

Brief description of the skill's purpose and capabilities.

## When to Use This Skill

- Trigger condition 1
- Trigger condition 2
- Trigger condition 3

## Instructions

[Step-by-step instructions for executing this skill]

## Examples

[Real-world examples showing the skill in action]
```
