---
description: Create a team of specialized agents based on the project analysis. Run /claude-army:analyze first. Generates agents in ~/.claude/agents/ with full subagent support.
tools: Read, Write, Glob, Grep, Bash
model: opus
---

Create a team of specialized Claude Code agents based on the forge analysis. Agents are written to `~/.claude/agents/` so they have full subagent spawning support.

## Before Starting

1. Read `.context/forge-analysis.md` — this contains the project analysis from `/claude-army:analyze`
2. If the file doesn't exist, tell the user to run `/claude-army:analyze` first
3. Read the templates in `${CLAUDE_PLUGIN_ROOT}/templates/` for agent archetypes

## Agent Creation Process

For each agent, read the matching template from `${CLAUDE_PLUGIN_ROOT}/templates/`, fill in project-specific details from the analysis, and write to `~/.claude/agents/`.

### Naming Convention

All agents use the prefix `<project>-`:
- `<project>-researcher.md`
- `<project>-builder.md`
- `<project>-reviewer.md`
- `<project>-challenger.md`
- `<project>-submitter.md`
- `<project>-orchestrator.md`
- `<project>-expert.md`
- `<project>-test-writer.md`

### What Goes Into Each Agent

**Researcher** — from analysis:
- What information the builder needs
- What verification is required
- Pre-checks (duplicates, open PRs)

**Builder** — from analysis:
- Code patterns section → exact templates
- Registration/integration pattern
- Build/generate commands

**Reviewer** — from analysis:
- Linter rules
- Test conventions
- PR checklist
- CI requirements

**Challenger** — from analysis:
- Lead reviewer name
- Tiered review patterns with exact quotes
- Things NOT to flag
- Fix routing to other agents

**Submitter** — from analysis:
- PR title format
- PR body structure
- Git workflow (branch naming, commit style)
- Checklist from PR template

**Orchestrator** — chains all agents:
- Phase 1: Spawn `<project>-researcher`
- Pause: User confirms research
- Phase 2: Spawn `<project>-builder`
- Phase 3: Run tests (self)
- Phase 4: Spawn `<project>-reviewer`, fix issues
- Phase 5: Spawn `<project>-challenger`, fix issues
- Pause: User confirms ready
- Phase 6: Spawn `<project>-submitter`
- Pause: User confirms push
- Must have `Agent` in tools list

**Expert** — from analysis:
- Architecture map
- Package purposes
- Entry points table

**Test Writer** — from analysis:
- Test package naming
- Assertion patterns
- Fixture styles
- Coverage target

### Grounding Rules

- Every code template must come from the actual project (read real files, don't use generic patterns)
- Every reviewer concern must be a real quote from the analysis
- Every test pattern must match the project's existing tests
- No hardcoded absolute paths — use project discovery
- All agents use `model: opus`

## Output

After creating all agents, show the user:
1. List of agents created with their paths
2. The orchestrator pipeline diagram
3. How to run: `@<project>-orchestrator [task description]`
4. Reminder that they can customize any agent by editing the file directly
