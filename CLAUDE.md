# CLAUDE.md — Claude Forge

> This is the starter CLAUDE.md. After running `/scope` → `/spec` → `/bootstrap`, this file will be replaced with a project-specific version.

## What is This?

Claude Forge is a universal project bootstrap system. It packages the best Claude Code workflow practices into a reusable template for any software project.

## Getting Started

Run these skills in order:

1. **`/scope`** — Interactive discovery conversation. Understand the idea, define the MVP, choose the stack. (~10-15 min)
2. **`/spec`** — Generate PROJECT_SPEC.md, ARCHITECTURE.md, PHASES.md from scope. (~5 min)
3. **`/bootstrap`** — Generate custom skills, hooks, rules, agents, and replace this CLAUDE.md with a project-specific version. (~2 min)

After bootstrap, start building:

4. **`/dev <feature>`** — Plan → build → verify loop
5. **`/wrap-up`** — End session cleanly

## Available Skills

### Phase A: Project Genesis (run once)
- `/scope` — Interactive project scoping
- `/spec` — Generate specification documents
- `/bootstrap` — Generate development infrastructure

### Phase B: Development (every session)
- `/dev` — Master development workflow
- `/wrap-up` — Session end ritual
- `/tdd` — Test-driven development
- `/debug` — Structured debugging
- `/learn` — Capture lessons
- `/phase-review` — Phase end audit + quality scoring
- `/skill-check` — Audit skill accuracy
- `/new-feature` — Generic feature creation (replaced by stack-specific skills after /bootstrap)

## Available Agents
- `@code-reviewer` — Architecture + code quality review
- `@test-writer` — Delegated test generation
- `@doc-keeper` — Documentation maintenance

<!-- BOOTSTRAP_PRESERVE_BEGIN: provocation -->
## Prompting Techniques

When output feels mediocre, use provocation prompts:
- "Knowing everything you know now, scrap this and implement the elegant solution"
- "Grill me on these changes — don't commit until you're satisfied"
- "Prove to me this works"
- "What would a senior engineer critique about this approach?"
- "Is there a simpler way that we're not seeing?"
<!-- BOOTSTRAP_PRESERVE_END: provocation -->

<!-- BOOTSTRAP_PRESERVE_BEGIN: context-management -->
## Context Management

- Name sessions descriptively: `phase1-auth-system`, `phase2-dashboard`
- One feature per session — don't context-switch
- At ~60% context: dump progress, `/wrap-up`, `/clear`, start fresh
- Don't push past 80% — quality degrades
- Delegate verbose operations to subagents
- Use `claude --worktree <name>` for parallel features
<!-- BOOTSTRAP_PRESERVE_END: context-management -->

<!-- BOOTSTRAP_PRESERVE_BEGIN: compaction -->
### Compaction Instructions

When compacting (`/compact`), always preserve:
- Current phase and session progress
- All file paths created or modified
- Architectural decisions made this session
- Test results (pass/fail)
- Open questions or blockers
- The current implementation plan
- Session handoff notes
<!-- BOOTSTRAP_PRESERVE_END: compaction -->

## Session Handoff

<!-- Updated by /wrap-up after each session -->
Last session: Not started — run /scope to begin
Next up: /scope → /spec → /bootstrap
Open questions: None
Known issues: None
