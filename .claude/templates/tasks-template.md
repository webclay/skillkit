# [Project Name] - Tasks

> **How this works:** Tasks are created dynamically as you request features. Claude updates this file as you build.

**Last Updated:** [Auto-updated by Claude]

---

## 🚧 In Progress

<!-- Current task appears here when you start working on something -->

---

## 📋 Pending

<!-- Tasks waiting to be worked on appear here -->

---

## ✅ Completed

<!-- Finished tasks are moved here with completion date -->

---

## 🐛 Bugs

<!-- Bug reports and fixes are tracked here -->

---

## Task Guidelines

### Status Icons
- 📋 **Pending** - Not started yet
- 🚧 **In Progress** - Currently working on
- ✅ **Completed** - Done and tested
- ⏸️ **Blocked** - Waiting on something
- 🐛 **Bug** - Issue to fix

### Task Structure

When Claude creates a task:

```markdown
**Task: [Task Name]**
- Status: 🚧 In Progress
- Priority: High / Medium / Low
- Dependencies: [Other tasks this depends on]
- Impacts: [Features this might affect - from projectbrief Feature Dependencies]

**Subtasks:**
- [ ] Subtask 1
- [ ] Subtask 2
- [ ] Subtask 3

**Notes:**
- [Implementation notes]
```

### How Tasks Are Created

1. **You request work** → "Build the payment settings page"
2. **Claude checks Feature Dependencies** in projectbrief.md
3. **Claude warns about impacts** → "This touches Authentication and Payments"
4. **Claude creates task** with subtasks and dependencies
5. **You work through subtasks** → Claude checks them off
6. **Run `/project:done`** → Task moves to Completed

---

## Related Files

- **Project overview:** `.claude/projectbrief.md`
- **Feature dependencies:** `.claude/projectbrief.md` (Feature Dependencies section)
- **Full PRD:** `/docs/prd.md`
