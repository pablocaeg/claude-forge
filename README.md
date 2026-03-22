# Claude Forge

A Claude Code plugin that reads your project and creates a team of specialized agents for it.

It looks at the codebase, studies how PR reviews work in the project, and generates agents that understand the project's actual conventions, not generic ones.

<br>

## What It Does

You get a small team of agents, each focused on one job, built from the project's real code and review patterns.

```mermaid
graph TD
    subgraph input [" What you provide "]
        A["Your task description"]
        B["Project codebase"]
        C["PR review access"]
    end

    subgraph forge [" Forge analyzes everything "]
        D["Code patterns\nand conventions"]
        E["Reviewer comments\nand priorities"]
        F["Test patterns\nand quality bar"]
    end

    subgraph agents [" Generated agent team "]
        G["Researcher"]
        H["Builder"]
        I["Reviewer"]
        J["Challenger"]
        K["Submitter"]
    end

    subgraph pipeline [" Orchestrated pipeline "]
        L["Research"] --> M["Build"] --> N["Review\nand fix"] --> O["Challenge\nand fix"] --> P["Submit"]
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

Reads the codebase and PR reviews. Extracts the lead reviewer's actual comments and ranks them by how often they come up. Saves the analysis to `.context/forge-analysis.md`.

### 3. Create the Agent Team

```
/claude-forge:create-team
```

Uses the analysis to generate agents in `~/.claude/agents/`. They live outside the plugin so they can spawn each other during orchestration.

### 4. Run the Pipeline

```
@[project]-orchestrator [describe your task]
```

Runs each agent in order with human checkpoints at key decisions.

### 5. Pre-check Before Submitting (optional)

```
/claude-forge:challenge
```

Simulates the lead reviewer's feedback on your current changes before you submit.

<br>

## What Gets Created

Every team is different. The forge reads the actual codebase and builds agents around its patterns.

| Agent | What it does | What makes it project-specific |
|-------|-------------|-------------------------------|
| **Researcher** | Gathers and verifies information | Knows what the builder needs, verifies against project requirements |
| **Builder** | Writes code | Uses templates from real project files, follows naming and structure |
| **Reviewer** | Checks against standards | Uses the project's linter config, test conventions, PR checklist |
| **Challenger** | Simulates the lead reviewer | Built from their actual comments, ranked by frequency |
| **Submitter** | Creates PRs | Matches the format from the project's best accepted PRs |
| **Orchestrator** | Runs the pipeline | Chains agents with test runs between phases and human approval gates |
| **Expert** | Answers codebase questions | Architecture map and entry points from the actual project |
| **Test Writer** | Writes tests | Uses the project's assertion library, fixture patterns, coverage target |

<br>

## Interesting Parts

### Reviewer Modeling

One thing I found useful while building this: reading through a project's PR reviews teaches you more than reading the source code. The forge captures that by extracting the lead reviewer's actual comments and building an agent that raises the same concerns before you submit.

```mermaid
graph LR
    subgraph collect [" Collect "]
        A["Read 20 to 40+\nPR reviews"]
    end

    subgraph extract [" Extract "]
        B["Lead reviewer\ncomments"]
    end

    subgraph rank [" Rank by frequency "]
        C["Tier 1: raised on\n50%+ of PRs"]
        D["Tier 2: raised on\n25 to 50%"]
        E["Tier 3: raised on\n10 to 25%"]
    end

    subgraph result [" Challenger agent "]
        F["Simulates review\nwith real quotes"]
        G["Routes each issue\nto the right agent"]
    end

    A --> B --> C & D & E --> F --> G

    style collect fill:#f0f4f8,stroke:#c9d6e3,color:#2d3748
    style extract fill:#ebf5ff,stroke:#90cdf4,color:#2b6cb0
    style rank fill:#fffbeb,stroke:#fbd38d,color:#975a16
    style result fill:#f0fff4,stroke:#9ae6b4,color:#276749
```

The idea is to catch likely feedback before submitting, which can save a few review rounds.

<br>

### Self-Verifying Research

I kept running into a problem where the research agent would confidently return wrong data, and that wrong data would end up in the code. So the research agent now verifies its own findings before passing them on.

| What it verifies | How |
|-----------------|-----|
| Algorithms | Computes step by step against known valid data |
| Sources | Fetches every URL to confirm it loads and has the right content |
| Facts | Cross-references from 2+ independent official sources |
| Status | Tags every finding: verified, partial, or unverified |

<br>

### Orchestrated Execution

Agents run in a managed pipeline, not independently. Each phase depends on the previous one. The orchestrator runs tests between phases and pauses for human approval at key moments.

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
        Note over O,R: Phase 1: Research
        O->>R: Gather and verify information
        R->>R: Cross-reference sources
        R->>R: Verify URLs
        R->>R: Prove algorithms step by step
        R-->>O: Findings with verification status
    end

    O-->>You: Research complete. Confirm?
    You->>O: Confirmed

    rect rgb(240, 248, 240)
        Note over O,T: Phase 2: Build and Test
        O->>B: Create code from findings
        B-->>O: Code ready
        O->>T: Write tests for new code
        T-->>O: Tests written
        O->>O: Run test suite
    end

    rect rgb(248, 245, 240)
        Note over O,V: Phase 3: Review and Fix
        O->>V: Check all changes against standards
        V-->>O: 2 blockers, 1 warning
        O->>B: Fix blocker in validation
        B-->>O: Fixed
        O->>T: Add missing test case
        T-->>O: Test added
        O->>O: Re-run tests
    end

    rect rgb(248, 240, 240)
        Note over O,C: Phase 4: Challenge and Fix
        O->>C: Simulate lead reviewer
        C-->>O: 3 questions with fix routing
        C-->>O: Q1 needs research, Q2 needs code fix
        O->>R: Verify Q1 claim
        R-->>O: Confirmed, no change needed
        O->>B: Fix Q2
        B-->>O: Fixed
        O->>O: Re-run tests
    end

    O-->>You: All checks pass. Submit?
    You->>O: Go ahead

    rect rgb(240, 240, 248)
        Note over O,S: Phase 5: Submit
        O->>S: Create PR
        S-->>You: PR URL
    end
```

<br>

## Things I Learned Building This

| Lesson | Details |
|--------|---------|
| Project-specific prompts matter | Generic agents produce generic output. Grounding them in the actual codebase makes a big difference. |
| PR reviews are the best teacher | Reading 20+ reviews taught the agents more than reading every source file. |
| Research needs verification | If the research agent gets something wrong, everything downstream is wrong too. |
| Restrict tools per agent | Read-only agents should not be able to write files. Fewer tools, fewer mistakes. |
| Humans should approve key decisions | The pipeline pauses after research and before submission. Full autonomy is tempting but risky. |
| Say what NOT to do | Telling agents what to avoid prevents mistakes better than only telling them what to do. |
| First version is always wrong | Test agents on real work, find what breaks, fix them. Repeat. |

<br>

## Project Structure

```
claude-forge/
├── .claude-plugin/
│   └── plugin.json            # Plugin manifest
├── skills/
│   ├── analyze/
│   │   └── SKILL.md           # /claude-forge:analyze
│   ├── create-team/
│   │   └── SKILL.md           # /claude-forge:create-team
│   └── challenge/
│       └── SKILL.md           # /claude-forge:challenge
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

- Works with Claude Code only. Other AI coding tools are not supported.
- Tested thoroughly on one project so far. The methodology works, but more testing would help.
- The forge produces a solid first draft of agents, but you will want to refine them after testing on real work.
- Not fully autonomous. You still review, approve, and guide the pipeline.

## Contributing

This is a work in progress. If you try it on a project and find ways to improve it, PRs and issues are welcome.

## License

MIT
