---
name: debugger
description: Systematic debugging for errors and unexpected behavior. Trigger words - debug, error, bug, not working, broken, fix, crash, exception, failed, issue, problem
---

# Debugger

Systematically investigates and fixes errors.

## When to Use This Skill

- User encounters an error message
- Something isn't working as expected
- User says "it's broken" or "this doesn't work"
- Build or runtime errors occur

## Instructions

### Step 1: Understand the Error

- Parse the error message for key information
- Identify the file and line number if provided
- Note the error type (runtime, syntax, type, network, etc.)

### Step 2: Gather Context

- Read the file where error occurs
- Check recent git changes (`git diff`)
- Look for related files (imports, dependencies)
- Search for similar patterns in codebase

### Step 3: Form Hypotheses

- List 2-3 likely causes based on error type
- Rank by probability
- Start with most likely cause

### Step 4: Investigate

- Check the suspected code
- Add console.log or debugging statements if needed
- Test the hypothesis

### Step 5: Fix and Verify

- Apply the fix
- Explain what caused the issue
- Verify the fix resolves the error

## Common Error Patterns

### TypeScript Errors
- "Cannot find module" → Check import path, file exists
- "Type X is not assignable to Y" → Check type definitions
- "Property does not exist" → Check object structure

### Runtime Errors
- "undefined is not a function" → Check if function exists, proper import
- "Cannot read property of undefined" → Add null checks
- "Network error" → Check API endpoint, CORS, auth

### Database Errors
- "Relation does not exist" → Run migrations
- "Unique constraint" → Check for duplicates
- "Foreign key violation" → Check related records exist

### React Errors
- "Hydration mismatch" → Server/client render differently
- "Invalid hook call" → Hook outside component
- "Too many re-renders" → Infinite loop in useEffect

## Output Format

```
### Root Cause
[Clear explanation of what's causing the error]

### Fix Applied
[What was changed and where]

### Why This Happened
[Brief explanation for the user to understand]

### Prevention
[How to avoid this issue in the future]
```

## Rules

- Be systematic, not random
- Show your reasoning process
- Fix the root cause, not just symptoms
- If stuck after 3 attempts, explain what was tried
- Don't make changes to unrelated code

## How to Verify

- Error no longer occurs
- App works as expected
- No new errors introduced
