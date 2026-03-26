# Full-Output Enforcement Guidelines

## Core Principle

"Treat every task as production-critical. A partial output is a broken output." The guidance prioritizes exhaustive delivery over brevity - if five components are requested, all five must be provided finished.

## Prohibited Shortcuts

The document bans common truncation patterns:
- Code: `// ...` or `/* ... */`
- Prose: "for brevity" or "and so on"
- Skeleton implementations
- Pattern-replacement descriptions

## Implementation Approach

Three steps:
1. Read the full request and count deliverables
2. Generate everything completely without partial drafts
3. Cross-check against the original scope before responding

## Token Limit Handling

When approaching capacity limits, stop at natural breakpoints rather than compressing content, then use a pause marker to resume cleanly without repetition.

The overall intent is ensuring users receive full, actionable outputs suitable for production use rather than abbreviated templates.
