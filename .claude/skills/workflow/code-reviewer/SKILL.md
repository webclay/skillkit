---
name: code-reviewer
description: Code review for quality, security, and maintainability. Trigger words - code review, review code, check code, security review, pr review, pull request review
---

# Code Reviewer

Reviews code for quality, security, and maintainability issues.

## When to Use This Skill

- After writing or modifying significant code
- Before making a commit
- User asks to "review this code" or "check my code"
- After completing a feature

## Instructions

### Step 1: Identify Changes

Run `git diff` to see recent changes, then focus review on modified files.

### Step 2: Review Checklist

#### Code Quality
- Code is clear and readable
- Functions and variables are well-named
- No duplicated code
- Proper error handling
- Follows existing patterns in the codebase

#### Security
- No exposed secrets or API keys
- Input validation on user data
- SQL injection prevention (parameterized queries)
- XSS prevention (proper escaping)
- Auth checks on protected routes

#### Performance
- No N+1 query problems
- Expensive operations not in loops
- Proper use of async/await
- No memory leaks (cleanup on unmount)

#### TypeScript
- Types are specific (avoid `any`)
- Null checks where needed
- Consistent naming conventions

### Step 3: Report Findings

Organize feedback by priority:

```
### Critical (must fix)
- [Issue with specific file:line reference]
- How to fix: [concrete solution]

### Warnings (should fix)
- [Issue]
- Suggestion: [improvement]

### Suggestions (consider)
- [Minor improvement ideas]
```

## Rules

- Be specific — reference exact file paths and line numbers
- Provide code examples for fixes when helpful
- Don't nitpick style issues if code works
- Focus on bugs and security over preferences
- If code looks good, say so briefly

## How to Verify

- All critical issues are addressed
- No security vulnerabilities in the reviewed code
- Code follows existing patterns in the codebase
