# Claude Forge

A Claude Code agent that helps you set up teams of specialized agents for your projects.

It reads your codebase, looks at how the project's PR reviews work, and creates agents tailored to the project's conventions — so you spend less time learning patterns and more time contributing.

<br>

## What It Does

Instead of one general-purpose AI assistant, the forge creates a small team of focused agents — each one grounded in the actual project's code, tests, and review expectations.

```mermaid
graph TD
    subgraph input [" What you provide "]
        A["Your task description"]
        B["Project codebase"]
        C["PR review access"]
    end

    subgraph forge [" Forge — analyzes everything "]
        D["Code patterns\n& conventions"]
        E["Reviewer comments\n& priorities"]
        F["Test patterns\n& quality bar"]
    end

    subgraph agents [" Generated agent team "]
        G["Researcher"]
        H["Builder"]
        I["Reviewer"]
        J["Challenger"]
        K["Submitter"]
    end

    subgraph pipeline [" Orchestrated pipeline "]
        L["Research"] --> M["Build"] --> N["Review\n& fix"] --> O["Challenge\n& fix"] --> P["Submit"]
    end

    A --> D
    B --> D
    B --> F
    C --> E

    D --> G & H
    E --> J
    F --> I

    G -.-> L
    H -.-> M
    I -.-> N
    J -.-> O
    K -.-> P

    style input fill:#f0f4f8,stroke:#c9d6e3,color:#2d3748
    style forge fill:#ebf5ff,stroke:#90cdf4,color:#2b6cb0
    style agents fill:#f0fff4,stroke:#9ae6b4,color:#276749
    style pipeline fill:#fffbeb,stroke:#fbd38d,color:#975a16
```

<br>

## Quick Start

### 1. Install

```bash
# As a Claude Code plugin
claude plugin install claude-forge --scope user

# Or manually
git clone https://github.com/pablocaeg/claude-forge.git
claude --plugin-dir claude-forge
```

### 2. Analyze the Project

```
/claude-forge:analyze
```

The forge reads the codebase, studies PR review patterns (extracts the lead reviewer's actual comments and ranks them by frequency), and saves a structured analysis to `.context/forge-analysis.md`.

### 3. Create the Agent Team

```
/claude-forge:create-team
```

Reads the analysis and generates a team of specialized agents in `~/.claude/agents/`, each grounded in the project's actual patterns. The agents live outside the plugin so they have full subagent spawning support.

### 4. Run the Pipeline

```
@[project]-orchestrator [describe your task]
```

The orchestrator chains all agents in sequence with human checkpoints at critical decisions.

### 5. Pre-check Before Submitting (optional)

```
/claude-forge:challenge
```

Simulates the lead reviewer's feedback on your current changes using the patterns extracted during analysis.

<br>

## What Gets Created

Every agent team is tailored to the specific project. The forge reads the actual codebase and designs agents grounded in its patterns — not generic templates.

| Agent | Purpose | How It's Customized |
|-------|---------|-------------------|
| **Researcher** | Gathers and verifies information | Knows what data the builder needs, verifies against project requirements |
| **Builder** | Creates code matching project conventions | Templates from actual project files, follows exact naming and structure |
| **Reviewer** | Checks against project standards | Uses the project's real linter config, test conventions, PR checklist |
| **Challenger** | Simulates the lead reviewer | Built from their actual PR comments, ranked by how often they raise each concern |
| **Submitter** | Creates polished PRs | Matches the format from the project's best accepted PRs |
| **Orchestrator** | Chains everything together | Dependencies between agents, test runs between phases, human approval gates |
| **Expert** | Answers codebase questions | Architecture map and entry points from the actual project |
| **Test Writer** | Writes tests matching conventions | Uses the project's assertion library, fixture patterns, coverage requirements |

<br>

## Interesting Parts

### Reviewer Modeling

One thing I found useful while building this: reading through a project's PR review access teaches you more than reading the source code. The forge tries to capture that by extracting the lead reviewer's actual comments and building an agent that raises similar concerns before you submit.

```mermaid
graph LR
    subgraph collect [" Collect "]
        A["Read 20–40+\nPR reviews"]
    end

    subgraph extract [" Extract "]
        B["Lead reviewer\ncomments"]
    end

    subgraph rank [" Rank "]
        C["Tier 1 — 50%+ of PRs"]
        D["Tier 2 — 25–50%"]
        E["Tier 3 — 10–25%"]
    end

    subgraph result [" Challenger agent "]
        F["Simulates review\nwith real quotes"]
        G["Routes issues\nto fix agents"]
    end

    A --> B --> C & D & E --> F --> G

    style collect fill:#f0f4f8,stroke:#c9d6e3,color:#2d3748
    style extract fill:#ebf5ff,stroke:#90cdf4,color:#2b6cb0
    style rank fill:#fffbeb,stroke:#fbd38d,color:#975a16
    style result fill:#f0fff4,stroke:#9ae6b4,color:#276749
```

> The idea is to catch likely review feedback before submitting, which can save a few round trips.

<br>

### Self-Verifying Research

I kept running into a problem where research agents would confidently return wrong data — and that wrong data would end up in the code. So the research agent now has a verification step: it cross-references sources and shows its work.

| Verification | How It Works |
|---|---|
| **Algorithms** | Computes step-by-step proofs against known valid data |
| **Sources** | Fetches every URL to confirm it's accessible and contains the claimed info |
| **Facts** | Cross-references from 2+ independent official sources |
| **Status** | Every finding tagged: Verified Verified, Partial Partial, Unverified Unverified |

<br>

### Orchestrated Execution

Agents run in a managed pipeline — not independently. Each phase depends on the previous one, with test runs between phases and human approval gates at key decisions.

```mermaid
sequenceDiagram
    actor You
    participant O as Orchestrator
    participant R as Researcher
    participant B as Builder
    participant T as Test Writer
    participant V as Reviewer
    participant C as Challenger
    participant S as Submitter

    rect rgb(240, 244, 248)
        Note over O,R: Phase 1 — Research
        O->>R: Gather & verify information
        R->>R: Cross-reference sources
        R->>R: Verify URLs accessible
        R->>R: Prove algorithms step-by-step
        R-->>O: Findings with verification status
    end

    O-->>You: Research complete — confirm?
    You->>O: Confirmed

    rect rgb(240, 248, 240)
        Note over O,T: Phase 2 — Build & Test
        O->>B: Create code from findings
        B-->>O: Code ready
        O->>T: Write tests for new code
        T-->>O: Tests written
        O->>O: Run test suite
    end

    rect rgb(248, 245, 240)
        Note over O,V: Phase 3 — Review & Fix
        O->>V: Check all changes against standards
        V-->>O: 2 blockers, 1 warning
        O->>B: Fix blocker in validation
        B-->>O: Fixed
        O->>T: Add missing test case
        T-->>O: Test added
        O->>O: Re-run tests
    end

    rect rgb(248, 240, 240)
        Note over O,C: Phase 4 — Challenge & Fix
        O->>C: Simulate lead reviewer
        C-->>O: 3 questions + fix routing
        C-->>O: Q1 needs research, Q2 needs code fix
        O->>R: Verify Q1 claim
        R-->>O: Confirmed — no change needed
        O->>B: Fix Q2 in invoices.go
        B-->>O: Fixed
        O->>O: Re-run tests
    end

    O-->>You: All checks pass — submit?
    You->>O: Go ahead

    rect rgb(240, 240, 248)
        Note over O,S: Phase 5 — Submit
        O->>S: Create PR
        S-->>You: PR URL
    end
```

<br>

## Things I Learned Building This

| Lesson | Details |
|--------|---------|
| **Project-specific prompts matter** | Generic agents produce generic output. Agents grounded in the actual codebase do much better. |
| **PR reviews are the best teacher** | Reading 20+ reviews taught the agents more than reading every source file. |
| **Research needs verification** | If the research agent gets something wrong, everything downstream is wrong. Self-verification helps. |
| **Restrict tools per agent** | Read-only agents shouldn't be able to write files. Fewer tools = fewer mistakes. |
| **Humans should approve key decisions** | The pipeline pauses after research and before submission. Fully autonomous is tempting but risky. |
| **Say what NOT to do** | Anti-patterns prevent the most common mistakes better than positive instructions alone. |
| **First version is always wrong** | Test agents on real work, find what breaks, improve. Repeat. |

<br>

## Project Structure

```
claude-forge/
├── .claude-plugin/
│   └── plugin.json            # Plugin manifest
├── skills/
│   ├── analyze/
│   │   └── SKILL.md           # /claude-forge:analyze — study the project
│   ├── create-team/
│   │   └── SKILL.md           # /claude-forge:create-team — generate agents
│   └── challenge/
│       └── SKILL.md           # /claude-forge:challenge — reviewer simulation
├── templates/                 # Agent archetypes used by create-team
│   ├── researcher.md
│   ├── builder.md
│   ├── reviewer.md
│   ├── challenger.md
│   ├── submitter.md
│   ├── orchestrator.md
│   ├── expert.md
│   └── test-writer.md
├── forge.md                   # Standalone agent (alternative to plugin)
├── docs/
│   ├── context-guide.md
│   ├── methodology.md
│   └── customization.md
├── LICENSE
└── README.md
```

<br>

## Requirements

| Requirement | Purpose |
|---|---|
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | Runs the plugin and agents |
| [GitHub CLI](https://cli.github.com/) (`gh`) | PR review analysis and submission |
| A project to work on | The forge needs a real codebase to analyze |

<br>

## Limitations

This is an experiment, not a production tool. Some honest caveats:

- **Claude Code only** — these agents don't work with other AI coding tools
- **One project tested thoroughly** — the methodology works, but more real-world testing is needed
- **Agents need iteration** — the forge produces a good first draft, but you'll want to refine after testing
- **Not truly autonomous** — you still need to review, approve, and guide the pipeline

## Contributing

This is a work in progress. If you try it on a project and find ways to improve it, PRs and issues are welcome.

## License

MIT
