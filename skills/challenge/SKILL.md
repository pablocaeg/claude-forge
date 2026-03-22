---
description: Run the challenger agent against your current changes to simulate the lead reviewer's feedback before submitting. Requires a forge analysis with reviewer patterns.
tools: Read, Glob, Grep, Bash, WebSearch, WebFetch
model: opus
---

Simulate the lead reviewer's feedback on the current changes. Uses reviewer patterns from the forge analysis.

## Before Starting

1. Read `.context/forge-analysis.md` for reviewer patterns
2. If it doesn't exist, tell the user to run `/claude-forge:analyze` first

## Process

1. Run `git status` and `git diff --name-only` to find all changes
2. Read every changed/new file
3. Apply reviewer checks in tier order (Tier 1 first, most likely to be raised)
4. For factual claims (thresholds, tax classification, etc.): WebSearch to verify
5. For source URLs: WebFetch to confirm accessibility

## Output

Present findings as the lead reviewer would — questions, not demands:

```
## Review Simulation: [Reviewer Name]

### Questions (likely to ask)
**file:line** — [question in their voice]
Fix with: [which agent or direct edit]

### Issues (would flag)
**file:line** — [what's wrong]
Fix with: [which agent]

### What Looks Good
[acknowledge well-done parts]

### Fix Plan
| # | Issue | Fix Agent | Instruction |
|---|-------|-----------|-------------|
| 1 | ... | ... | ... |

Execution order: [dependencies]
```
