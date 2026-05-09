---
name: "wrap-up"
description: "Session end ritual — commit, capture lessons, write handoff for next session."
---

# /wrap-up — Session End Ritual

End-of-session skill that captures work, lessons, and state for the next session.

## Branch: are we inside a kanban-spawned worktree?

**First action: check `KANBAN_CARD_ID`.**

```bash
echo "${KANBAN_CARD_ID:-}"
```

- **If `KANBAN_CARD_ID` is set** → run the **Kanban Card Completion path** (skip to "Kanban Card Completion" section below). The kanban app aggregates per-card state; this session must not stomp on the parent project's `transfer-context.json` or root `CLAUDE.md` Session Handoff section.
- **If `KANBAN_CARD_ID` is empty** → run the standard workflow below (Steps 1–8). This is a normal single-session wrap-up.

The env var is set by `/kanban-prep` in the worktree setup commands; if a user ran `claude` directly inside a worktree without it, no protection — the standard flow runs and may overwrite the parent state. The convention is documented in every card brief.

---

## Workflow (standard — single-session)

### 1. REVIEW WHAT CHANGED
```bash
git diff --stat
git log --oneline -10
```

Summarize:
- Which files were created
- Which files were modified
- How many lines changed
- Any major features completed

### 2. CAPTURE LESSONS
For each lesson learned in this session:
- Call `/learn` to save it to CLAUDE.md
- `/learn` handles deduplication automatically
- Focus on: non-obvious gotchas, anti-patterns, decisions, traps

Example lessons:
- "Don't use X for Y because it causes Z"
- "When debugging A, check B first (we missed this and it cost time)"
- "The API for Z changed in version N — use W instead"

### 3. STAGE & COMMIT WORK
**Before committing, ask user approval:**

> "I'm about to commit the following changes:
> - Created: [files]
> - Modified: [files]
> - Commit message: [message]
>
> Approve?"

Once approved:
```bash
git add .
git commit -m "[Phase N] Feature X: [summary]. [Provocation note]."
```

Example commit:
```
[Phase 2b] User search API: GET /api/users/search with pagination. Considered caching but decided against it for now (simpler, can optimize later).
```

### 4. UPDATE PHASES.md
Record progress in `PHASES.md`:
- Which tasks were completed
- Which tasks remain
- Any blockers or decisions
- Estimated progress (%)

Format:
```markdown
## Phase 2b: Development
- [x] Task 1
- [x] Task 2
- [ ] Task 3 (blocked: awaiting decision from user)
- [ ] Task 4

Progress: 65%
```

### 5. WRITE HUMAN-READABLE HANDOFF
Update `CLAUDE.md` at project root with:

```markdown
## Current Status
- Phase: [N]
- Last session: [date]
- Focus: [what we were building]
- Status: [complete, in progress, blocked]

## What's Done
- [feature 1]
- [feature 2]
- [feature 3]

## What's Next
- [feature 4]
- [feature 5]

## Open Decisions
- [decision 1]: context needed
- [decision 2]: awaiting user input

## Lessons Learned (this session)
- [lesson 1]
- [lesson 2]
```

### 6. WRITE MACHINE-READABLE HANDOFF
Create/update `.claude/transfer-context.json`:

```json
{
  "date": "2026-03-19T15:45:00Z",
  "phase": "2b",
  "completed_work": [
    "Built user search API endpoint",
    "Added pagination support",
    "Wrote integration tests"
  ],
  "files_created": [
    "src/api/users.ts",
    "tests/api/users.test.ts"
  ],
  "files_modified": [
    "src/routes.ts",
    "src/db/queries.ts"
  ],
  "open_decisions": [
    {
      "decision": "Cache user search results?",
      "context": "Performance is fine now, but may need it at scale",
      "suggested_by": "user"
    }
  ],
  "traps_to_avoid": [
    "API endpoint needs URL encoding for special chars (we fixed this)",
    "Database queries return NULL for empty results (remember to handle)"
  ],
  "current_branch": "feature/user-search",
  "test_status": "12 passed, 0 failed",
  "next_tasks": [
    "Build search filters UI",
    "Add rate limiting",
    "Write documentation"
  ]
}
```

### 7. CONTEXT MANAGEMENT
Check context usage:
- If context is getting high (>80% used), suggest `/compact`
- `/compact` removes old conversation turns but keeps lessons and state

### 8. SUGGEST SESSION NAMING
Suggest to user:
> "This session accomplished: [summary]. Suggest naming this session: 'user-search-api' or 'phase2b-apr1'?"

## Checklist
- [ ] Reviewed git changes
- [ ] Called `/learn` for each lesson
- [ ] Staged and committed with user approval
- [ ] Updated PHASES.md
- [ ] Updated root CLAUDE.md (human-readable)
- [ ] Updated .claude/transfer-context.json (machine-readable)
- [ ] Checked context usage
- [ ] Suggested session name

## Example Session End
```
[User completes feature]

SKILL: "Session wrap-up time! Let me capture what we've done."

SKILL: "Changes:
- Created: src/search.ts, tests/search.test.ts
- Modified: src/routes.ts, src/db.ts
- 250 lines added, 15 deleted
- Tests: all green"

SKILL: "Capturing lessons... [calls /learn 3 times]"

SKILL: "Ready to commit?
- Created: [files]
- Modified: [files]
- Message: '[Phase 2b] User search: GET /api/users/search with pagination'"

USER: "Looks good"

SKILL: [commits]

SKILL: [updates PHASES.md, CLAUDE.md, transfer-context.json]

SKILL: "Context usage: 68%. You're good.

Session captured. Next time you'll resume with:
- Phase: 2b
- Last work: user search API
- Next: search filters UI
- Open decision: caching strategy

Suggested session name: 'user-search-phase2b'"
```

## Kanban Card Completion (when `KANBAN_CARD_ID` is set)

This path runs when the session is a kanban-spawned worktree handling a single card. The standard handoff steps are replaced with per-card reporting; the parent project's `transfer-context.json` and root `CLAUDE.md` are not touched.

### 1. Review what changed (worktree-scoped)

```bash
git diff --stat
git log --oneline -10
```

### 2. Capture lessons (per-card)

Write lessons into the per-card report (Step 5 below), not into the parent `CLAUDE.md`. The card-completion aggregator will roll them up at phase-end.

### 3. Stage & commit work in the worktree

Same as the standard flow — get approval, then commit with TDD-aware message including `useful-tests-audit:` block.

### 4. Run the Useful Tests Gate one more time

Before declaring the card done, confirm the audit was performed for every commit on this worktree's branch (`git log --grep="useful-tests-audit:"`). If any commit lacks it, write a remediation note in the card report.

### 5. Write per-card completion report

Path: `.claude/kanban/completed/<KANBAN_CARD_ID>.md` (relative to **the worktree**, since this is the agent's working tree).

```markdown
# Card: <KANBAN_CARD_ID>

**Title**: <from manifest>
**Phase**: <from manifest>
**Branch**: <git current branch>
**Worktree path**: <pwd>
**Completed at**: <ISO timestamp>

## What was built
- <bullet 1>
- <bullet 2>

## Files changed
<git diff --stat output>

## Useful Tests Audit
- All commits on this branch include `useful-tests-audit:` block: ✅ / ❌
- Tests added: <count>
- TDD skips with reason: <count>
- Useful-tests-skip overrides: <count>

## Lessons (for parent rollup)
- <lesson 1>
- <lesson 2>

## Open issues / blockers
- <if any>

## Acceptance criteria
- [x] <each criterion from card brief>
- [ ] <unmet, with reason>

## Ready to merge
- [ ] All acceptance criteria met
- [ ] Tests green
- [ ] Useful Tests Gate passed for every commit
- [ ] No conflicts with `main` (`git merge-base` is recent)
```

### 6. DO NOT touch parent state

Skip the standard wrap-up's:
- ❌ Updating root `CLAUDE.md` Session Handoff section
- ❌ Updating root `.claude/transfer-context.json`
- ❌ Updating root `PHASES.md`

These are owned by the parent project. The kanban aggregation step (run after all cards in a phase complete) reads `.claude/kanban/completed/*.md` and rolls them up into the parent state in one consolidated update.

### 7. Suggest the merge action

Present to the user:

> Card `<id>` complete. To merge:
>
> ```bash
> # From the parent project root:
> git fetch
> git checkout main && git pull
> git merge --no-ff kanban/<id>
> git worktree remove <worktree-path>
> ```
>
> Or in Cline Kanban: move the card to "Done" / trash. Linked downstream cards will auto-start.

---

## Key Principles
1. **Ask before committing**: Always get user approval
2. **Lessons matter**: Every session should add 1–3 lessons to the knowledge base
3. **State is precise**: `transfer-context.json` must be accurate (next session depends on it)
4. **Human and machine**: `CLAUDE.md` for humans, `transfer-context.json` for agents
5. **Traps to avoid**: Document things that took time or were non-obvious
6. **Kanban-aware**: When `KANBAN_CARD_ID` is set, write per-card reports, never stomp on parent state
