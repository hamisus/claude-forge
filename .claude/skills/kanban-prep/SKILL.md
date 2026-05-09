---
name: kanban-prep
description: Optional — export parallel-safe phase tasks as kanban-importable card briefs. Reads PHASES.md and emits per-card markdown briefs + a JSON manifest suitable for Cline Kanban (paste-import) or any other kanban tool.
version: 1.0.0
author: Claude Forge
category: phase-b
phase: build
---

# /kanban-prep — Export Parallel-Safe Tasks as Kanban Cards

> **Optional skill.** Use when you want to run a phase's work in parallel via [Cline Kanban](https://github.com/cline/kanban) (or any other kanban tool). Skipping it is fine — `/dev` runs single-threaded by default.

## What it does

Reads `PHASES.md` (annotated with `parallel-safe` / `depends-on` per `/spec`), picks the parallel-safe items in the current phase, and writes:

1. **`.claude/kanban/manifest.json`** — machine-readable index of all generated cards.
2. **`.claude/kanban/cards/<id>.md`** — one per card, paste-ready into a kanban tool's UI. Each brief encodes what the agent in that worktree needs to know.

It does **not** auto-create cards in any kanban tool. The contract is the markdown file — copy/paste into Cline Kanban, Linear, GitHub Projects, Trello, or wherever you track work. (If a Cline Kanban MCP server becomes available later, this skill detects it and offers programmatic creation as a bonus path.)

## Prerequisites

- `/scope` → `/spec` → `/bootstrap` complete.
- `PHASES.md` in `.claude/specs/` has `parallel-safe` / `depends-on` annotations on its work items (added by `/spec`).

## Usage

```
/kanban-prep                  # exports current phase's parallel-safe items
/kanban-prep --phase 3        # exports phase 3 specifically
/kanban-prep --all            # exports every parallel-safe item across all phases
```

## Process

### 1. Identify candidates

Parse `PHASES.md` for the target phase. Pick work items where:

- `parallel-safe: true`, AND
- All `depends-on` items are either complete (per `PHASES.md` checkbox) or also being exported in this run with no circular dependency.

Items that are `parallel-safe: false` or have unresolved dependencies stay in the sequential queue (handled by `/dev`).

### 2. Pull useful-test contracts

For each card, look up the parent feature in `.claude/specs/PROJECT_SPEC.md`. Extract the feature's **Useful Test Outline** entries that apply to this work item. These ride into the card brief — the worktree agent uses them as the RED-phase contracts.

### 3. Write `.claude/kanban/manifest.json`

```json
{
  "generated_at": "<ISO 8601>",
  "phase": "<phase-id>",
  "cards": [
    {
      "id": "phase-2.api-scaffold",
      "title": "Backend: scaffold user-search API",
      "phase": "2",
      "depends_on": [],
      "parallel_safe": true,
      "estimated_units": 2,
      "status": "ready",
      "worktree_name": "kanban/phase-2.api-scaffold"
    }
  ]
}
```

### 4. Write `.claude/kanban/cards/<id>.md` (per card)

Template:

```markdown
# {{title}}

**Card ID**: `{{id}}`
**Phase**: {{phase}}
**Depends on**: {{depends_on or "none"}}
**Parallel-safe**: {{parallel_safe}}
**Estimated units**: {{estimated_units}}

## Goal

{{1–2 sentences from PHASES.md task description, expanded with surrounding feature context from PROJECT_SPEC.md.}}

## Acceptance criteria

- [ ] {{from PROJECT_SPEC.md feature Success Criteria}}
- [ ] {{from PHASES.md Verification Checklist}}
- [ ] All commits include a `useful-tests-audit:` block (per `.claude/rules/useful-tests.md`)
- [ ] Branch `kanban/{{id}}` merges into `main` cleanly

## Useful test expectations

These come from the feature's **Useful Test Outline** in PROJECT_SPEC.md. They are the contracts the worktree must satisfy:

- {{test contract 1}}
- {{test contract 2}}
- {{test contract 3}}

## Forge skills to use in this worktree

- **`/dev`** — runs the unit through TDD-default loop
- **`/tdd`** — RED → GREEN → REFACTOR cycle (entered automatically by `/dev`)
- **`@code-reviewer`** — call before declaring the card done
- **`/wrap-up`** — when complete (will auto-detect `KANBAN_CARD_ID` and write per-card report)

## Worktree setup

Run from the parent project root:

```bash
# 1. Create the worktree
git worktree add ../{{project_slug}}-{{id}} -b kanban/{{id}}

# 2. Enter it
cd ../{{project_slug}}-{{id}}

# 3. Install deps if not symlinked (kanban tools usually symlink gitignored dirs)
{{install_command}}

# 4. Launch Claude Code with the card ID
KANBAN_CARD_ID={{id}} claude
```

The `KANBAN_CARD_ID` env var tells `/wrap-up` to write a per-card report instead of touching parent state. Don't forget it.

## When you're done

1. Verify acceptance criteria above.
2. Run `/wrap-up` — it will write `.claude/kanban/completed/{{id}}.md` and skip parent-state updates.
3. Merge from the parent project (see `/wrap-up` output for exact commands), or move the card to "Done" in your kanban tool to trigger any auto-merge / linked-card start.
```

### 5. Print summary

```
✓ Generated 4 card briefs for phase 2:
  - .claude/kanban/cards/phase-2.api-scaffold.md
  - .claude/kanban/cards/phase-2.ui-shell.md
  - .claude/kanban/cards/phase-2.search-endpoint.md
  - .claude/kanban/cards/phase-2.search-ui.md
✓ Manifest at .claude/kanban/manifest.json

3 items remain sequential (depends-on chain or parallel-safe: false).

Next:
1. Open each .md, copy title + body into your kanban tool.
2. Or, if Cline Kanban is running locally:
   npx kanban  (from parent project root)
   Then drag-drop the card files into the board.
3. For each card, run the worktree setup commands at the bottom of the brief.
```

## Detecting an installed Cline Kanban MCP

If the agent has access to MCP tools matching `mcp__kanban__*` (or similar), `/kanban-prep` offers a programmatic create path:

> "Detected Cline Kanban MCP. Want me to create the cards programmatically? (y/N)"

Default is **no** — the markdown files are always written first; programmatic create is a bonus that may be reverted if it fails. We don't depend on this API existing.

## What this skill does NOT do

- Does **not** spawn worktrees or launch agents — that's the kanban tool's job.
- Does **not** track card status — the kanban tool owns that.
- Does **not** merge worktrees back — that's `/wrap-up`'s suggested command (or kanban auto-commit).
- Does **not** replace `/dev` for sequential work — items not parallel-safe still flow through `/dev` normally.

## Risks and known issues

- **Cline Kanban is research preview** with a recent CVE (websocket hijacking from malicious sites; patched, but worth knowing). Don't browse arbitrary sites with the kanban server running.
- **Symlink-based gitignored sharing** breaks if a worktree modifies gitignored files. Avoid that pattern in cards.
- **Merge conflicts** are still a real concern even when items are tagged parallel-safe. The tag means "no shared file dependencies on paper" — runtime can surprise. Prefer cards that touch fully disjoint directories.
- **`/dev` inside a worktree** works the same way (TDD-default + useful-tests gate); the only behavioral difference is `/wrap-up` writes per-card output.

## Output convention recap

| Path | Owner | When written |
|------|-------|-------------|
| `.claude/kanban/manifest.json` | `/kanban-prep` | At export time |
| `.claude/kanban/cards/<id>.md` | `/kanban-prep` | At export time |
| `.claude/kanban/completed/<id>.md` | `/wrap-up` (in worktree) | When card is done |
| `KANBAN_CARD_ID` env var | User (via card brief setup commands) | At worktree launch |
