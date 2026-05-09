---
name: "dev"
description: "Master development workflow — plan, build, verify, iterate on any feature. Stack-agnostic: works for any language/framework."
---

# /dev — Development Workflow

Core skill for implementing features in Phase B. Runs every session.

## Workflow

### 1. READ SESSION CONTEXT
- Read `CLAUDE.md` for session handoff and accumulated lessons
- Read `.claude/transfer-context.json` for machine-readable state
- Read `PHASES.md` to confirm current phase
- Absorb any project-specific patterns, gotchas, anti-patterns

### 2. UNDERSTAND THE FEATURE REQUEST
- What is the user asking to build?
- What does "done" look like?
- Any constraints (performance, compatibility, design)?
- Any lessons from CLAUDE.md that apply?

### 3. PRODUCE A PLAN (Annotation Cycle)
Use the Annotation Cycle for iterative planning:

**Step 1: Produce**
- Break the feature into small logical units (3–5 items)
- For each unit, define: what, why, any risks
- Example:
  ```
  ## Plan
  1. Create API endpoint for user search (GET /api/users/search)
     - Validates query param length
     - Returns paginated results
  2. Add database query function
     - Uses prepared statements
     - Handles empty results
  3. Write integration tests (RED phase of TDD)
  4. Implement endpoint handler
  5. Verify tests pass
  ```

**Step 2: Guard Phrase**
Present the plan to the user with:
> "Here's the plan. Review and annotate before I implement."

**Step 3: Wait for Annotations**
- User may ask: "Can we do step 1 differently?"
- User may flag: "This will break X"
- User may add: "Also make sure to handle Y"

**Step 4: Revise**
- Update plan based on feedback
- If significant changes: loop back to Step 1 (produce new plan)
- If minor tweaks: confirm and proceed

### 4. BUILD IN SMALL LOGICAL UNITS — TDD-DEFAULT

Once the plan is approved, every unit defaults to the TDD cycle (RED → GREEN → REFACTOR). This is a default, not a wall — see "Override path" below.

**For each unit:**

1. **Read the Useful Test Outline** for the parent feature in `.claude/specs/PROJECT_SPEC.md`. These are the test contracts the feature owes. The outline drives the RED phase.
2. **Enter `/tdd`** — write failing tests first, then implementation. See `.claude/skills/tdd/SKILL.md` for the cycle.
3. **Run the Useful Tests Gate** (Step 5 below) before committing. This audits whether the tests you just wrote are worth keeping.
4. **Commit with descriptive message** including the `useful-tests-audit:` block from the gate.
5. **Show user the result.**

Use stack-specific skills when available (`/new-component`, `/db-schema`, etc.) — they typically run inside `/tdd`'s GREEN phase, generating the implementation that makes tests pass.

**Override path** — if TDD genuinely doesn't fit (one-off scripts, config edits, design exploration):

- User says: "skip TDD for this unit" with a reason.
- Agent records `tdd-skip-reason: <reason>` in the commit footer **and** appends to `.claude/transfer-context.json` `tdd_skips[]`.
- **Silent skips are forbidden.** No reason → run TDD.
- Common acceptable reasons: `"config-only edit, no behavior to test"`, `"throwaway exploration script"`, `"renaming a file with no logic change"`. Less acceptable, will surface at phase-review: `"too hard to test"` (write a smoke test instead), `"will add tests later"` (no — write them now or log the skip honestly).
- Phase-review surfaces the skip-rate. >20% triggers a discussion.

### 5. USEFUL TESTS GATE

After GREEN, before commit, audit the new/changed tests against `.claude/rules/useful-tests.md`. Three steps:

1. **Mechanical scan** — grep for smell patterns (no `expect`/`assert`, only `toHaveBeenCalled`, snapshot-only, mocks of SUT, vague titles like `"works"`).
2. **Coverage map** — for each new/changed source unit, confirm at least one test references it. Flag zero-reference units.
3. **Judgment pass (mandatory)** — for each new test, name **one plausible mutation** that would not be caught (`returned wrong type`, `off-by-one`, `swapped branches`, `stale cache`, `threw instead of returned`, etc.). Document in commit body under `useful-tests-audit:` (one line per test).

If the gate fails (suspect tests not addressed, mutations not named, or a test admits it would not catch an obvious mutation):

- **Strengthen the test** until it would catch the mutation, OR
- Log `useful-tests-skip-reason: <reason>` in the commit footer (use sparingly).

**The audit lives in the commit message**, not in a separate file. This makes the rubber-stamp risk visible: vague answers in commit history are themselves a smell that phase-review will catch.

### 6. VERIFICATION & QUALITY
After each unit:
- Check for build errors (hooks run automatically)
- Check for linting (hooks run automatically)
- Run tests (hooks run automatically)
- If red/broken: debug immediately using `/debug` skill
- If green: move to next unit

**Provocation Check**
Before writing a unit, ask:
- "Is there a simpler way to do this?"
- "What could go wrong here?"
- "Are we overcomplicating this?"

Document the answer in the commit message.

### 7. OPTIONAL: SUBAGENT REVIEW
If available and appropriate:
- Tag `@code-reviewer` for complex logic
- Tag `@db-reviewer` for schema/queries
- Cite their feedback in commits

### 8. ITERATION
If plan changes mid-build:
- Update the plan in chat
- Get user approval for new units
- Commit the delta
- Continue

### 9. WRAP UP
When all units are complete:
- Verify all tests pass
- Run any final integration tests
- Suggest `/wrap-up` to capture lessons and commit session

## Anti-Patterns
- ❌ Skip the plan — you'll build wrong things
- ❌ Modify tests mid-build — tests define the contract
- ❌ **Skip TDD without logging a `tdd-skip-reason`** — silent skips are forbidden
- ❌ **Write tautological tests to satisfy the gate** — the audit is judged at phase-review; rubber-stamping shows up there
- ❌ **Use `useful-tests-skip-reason` to bypass the audit** when the real fix is a better test
- ❌ Leave broken code mid-stream — fix it before moving on
- ❌ Assume stack without asking — always check available skills
- ❌ Burn context on random tries — debug systematically using `/debug`

## Integration with Other Skills
- **`/tdd`**: Default cycle for every unit. Override only with logged reason.
- **`/debug`**: Use when something breaks mid-build
- **`/learn`**: Use to capture gotchas and lessons
- **Stack-specific skills**: Use instead of implementing directly (e.g., `/new-component` for React) — typically inside `/tdd` GREEN phase
- **Rules**: `.claude/rules/useful-tests.md` defines the gate criteria

## Example Session
```
USER: "Build a user search endpoint"

SKILL: [reads CLAUDE.md, transfer-context.json, PHASES.md]
SKILL: "OK, I see we're using [language/framework]. Let me check if there's a stack-specific skill for this..."

SKILL: [produces plan with 5 units]
SKILL: "Here's the plan. Review and annotate before I implement."

USER: "Unit 3 should also validate against a blocklist"
SKILL: [updates plan]
SKILL: "Here's the updated plan. Approved?"

USER: "Go ahead"
SKILL: [implements unit 1, commits, shows result]
SKILL: [implements unit 2, commits, shows result]
[... and so on]
SKILL: "All units complete. Tests: 12 passed. Shall I run /wrap-up?"
```

## Key Principles
1. **Stack-agnostic**: Describe patterns, not specific commands
2. **Small units**: Commit frequently, verify often
3. **Annotation cycle**: Get user approval before implementing
4. **Hooks handle verification**: Don't manually run linters/tests (hooks do it)
5. **Lessons matter**: Capture gotchas via `/learn`
6. **Provocation**: Always ask "Is there a simpler way?"
