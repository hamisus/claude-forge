# Claude Forge

A universal project bootstrap for Claude Code. Go from idea to fully-configured development environment in one session.

## What It Does

Claude Forge packages the best Claude Code workflow practices into a reusable template:

- **12 skills** that orchestrate the entire development lifecycle
- **3 specialized agents** for code review, test writing, and doc maintenance
- **Automated hooks** for quality gates, context preservation, and notifications
- **Self-improving system** that gets smarter about your project over time

## Quick Start

```bash
# Clone for a new project
git clone https://github.com/hamisus/claude-forge my-project
cd my-project
git init  # Fresh git history for your project

# Open Claude Code
claude

# Phase A: Project Genesis (one-time, ~20 minutes)
/scope          # Interactive discovery conversation
/spec           # Generate spec + architecture docs
/bootstrap      # Generate stack-specific skills, hooks, rules

# Phase B: Development (every session)
/dev <feature>  # Plan → build → verify loop
/wrap-up        # End session cleanly
```

## How It Works

### Phase A: Genesis

**`/scope`** conducts an interactive conversation to understand your project — what you're building, who it's for, what tech to use, and what the MVP looks like. Outputs a structured scope document.

**`/spec`** takes the scope and generates four specification documents: PROJECT_SPEC.md, ARCHITECTURE.md, PHASES.md, and DESIGN_SYSTEM.md (for UI projects).

**`/bootstrap`** reads the specs and generates everything Claude Code needs: a custom CLAUDE.md, stack-specific skills (e.g., `/new-component` for React, `/new-screen` for Flutter), hooks (linter, test runner, code gen), path-specific rules, and tailored review agents.

### Phase B: Development

Every session follows the same loop:

```
/dev <feature>     Plan → build → verify → report
  ↳ Uses /tdd     For test-first development
  ↳ Uses /debug   When something breaks
  ↳ Uses /learn   When something non-obvious is discovered
/wrap-up           Commit, capture lessons, write handoff
```

At the end of each phase:
```
/phase-review      Audit + quality score (nothing ships below 80/100)
/skill-check       Verify skills are still accurate
```

## What's Included

```
claude-forge/
├── CLAUDE.md                     # Starter file (replaced by /bootstrap)
├── DESIGN.md                     # System design philosophy
├── README.md                     # This file
│
├── .claude/
│   ├── settings.json             # Universal hooks + permissions
│   │
│   ├── skills/
│   │   ├── scope/SKILL.md        # Project scoping conversation
│   │   ├── spec/SKILL.md         # Specification generator
│   │   ├── bootstrap/SKILL.md    # Infrastructure generator
│   │   ├── dev/SKILL.md          # Development workflow
│   │   ├── wrap-up/SKILL.md      # Session end ritual
│   │   ├── tdd/SKILL.md          # Test-driven development
│   │   ├── debug/SKILL.md        # Structured debugging
│   │   ├── learn/SKILL.md        # Lesson capture
│   │   ├── phase-review/SKILL.md # Phase audit + scoring
│   │   ├── skill-check/SKILL.md  # Meta-skill health
│   │   └── new-feature/SKILL.md  # Generic feature template
│   │
│   ├── agents/
│   │   ├── code-reviewer.md      # Architecture + quality review
│   │   ├── test-writer.md        # Test generation
│   │   └── doc-keeper.md         # Documentation maintenance
│   │
│   └── rules/
│       └── orchestrator-protocol.md  # Contractor-mode loop
│
└── docs/
    └── workflow-guide.md          # Quick reference
```

## Key Concepts

**Annotation Cycle**: Plans are reviewed iteratively. Claude presents a plan → you annotate → Claude revises → repeat until approved. Prevents wasted work.

**Guard Phrases**: Specific phrases that keep Claude disciplined: "don't implement yet", "verify before proceeding", "plan this before coding".

**Quality Gates**: PreCommit hooks run linter + tests. Phase reviews score on 5 dimensions (100 points). Nothing ships below 80.

**Context Hygiene**: Session handoff (human-readable in CLAUDE.md + machine-readable in transfer-context.json) ensures seamless transitions. Compact at 60%, clear at 80%.

**Self-Improvement**: `/learn` captures lessons → `/skill-check` audits skills → `/phase-review` scores quality → system gets smarter every session.

## Works With Any Stack

Claude Forge is fully stack-agnostic. The core skills work for any language or framework. During `/bootstrap`, stack-specific skills, hooks, and rules are generated based on your project's actual tech choices.

Tested with: Flutter/Dart, React/Next.js, Python/FastAPI, Go, Rust, Ruby on Rails, and more.

## License

MIT
