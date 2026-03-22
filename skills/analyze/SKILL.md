---
description: Analyze a project's codebase, PR review patterns, and conventions. Run this first before creating agents. Outputs a structured analysis that create-team uses.
tools: Read, Glob, Grep, Bash, WebSearch, WebFetch
model: opus
---

Analyze the current project to prepare for agent team creation. Read everything relevant and produce a structured analysis.

## Step 1: Project Foundation

Read these files if they exist:
- README.md, CONTRIBUTING.md, CHANGELOG.md
- .github/pull_request_template.md
- .github/ISSUE_TEMPLATE/

## Step 2: Tech Stack

Identify:
- Language and version (go.mod, package.json, Cargo.toml, pyproject.toml, etc.)
- Build system and commands
- Linter config with exact rules
- Test framework, assertion library, coverage requirements
- CI/CD workflows (.github/workflows/)

## Step 3: Code Patterns

- Map directory structure with Glob
- Read 5-10 key source files for patterns
- Read 5-10 test files for testing conventions
- Identify the extension/plugin/registration pattern (how new features are added)

## Step 4: PR Review Analysis

Find the lead reviewer:
```bash
gh pr list --state all --limit 100 --json number,reviews --jq '.[].reviews[].author.login' 2>/dev/null | sort | uniq -c | sort -rn | head -5
```

Find external contributor PRs:
```bash
gh pr list --state all --limit 200 --json number,title,state,author --jq '.[] | "\(.number) [\(.state)] \(.author.login): \(.title)"' 2>/dev/null
```

For each external PR with lead reviewer feedback, extract BOTH:

PR-level reviews:
```bash
gh api repos/OWNER/REPO/pulls/NUMBER/reviews --jq '.[] | select(.user.login == "REVIEWER") | .body' 2>/dev/null
```

Inline code comments:
```bash
gh api repos/OWNER/REPO/pulls/NUMBER/comments --jq '.[] | select(.user.login == "REVIEWER") | "FILE: \(.path):\(.line // .original_line)\nCOMMENT: \(.body[:400])"' 2>/dev/null
```

Analyze at least 15-20 PRs. Rank reviewer patterns by frequency into tiers.

## Step 5: Context Folder

Read everything in `.context/` if it exists.

## Output

Save the analysis to `.context/forge-analysis.md` with these sections:

```markdown
# Forge Analysis: [Project Name]

## Tech Stack
[language, build, linter, test framework, CI]

## Code Patterns
[file structure, naming conventions, registration pattern]

## Test Conventions
[package naming, assertion library, fixture patterns, coverage requirement]

## PR Review Patterns
### Lead Reviewer: [name]
### Tier 1 (50%+ of PRs)
[patterns with exact quotes]
### Tier 2 (25-50%)
[patterns with exact quotes]
### Tier 3 (10-25%)
[patterns with exact quotes]
### Things NOT to Flag
[verified false positives]

## PR Format
[title format, body structure, checklist]

## Contribution Type
[what external contributors typically add]

## Build/Generate Commands
[commands that must run before submitting]
```

Tell the user the analysis is complete and they can run `/claude-forge:create-team` next.
