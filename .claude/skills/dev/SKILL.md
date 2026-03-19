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

### 4. BUILD IN SMALL LOGICAL UNITS
Once plan is approved:

- Implement ONE unit at a time
- Commit after each unit (descriptive message)
- Use stack-specific skills when available (`/new-component`, `/db-schema`, etc.)
- If stack-specific skill exists: delegate to it instead of implementing directly

**For each unit:**
1. Write code
2. Run verification (hooks handle linting/testing)
3. Commit with clear message
4. Show user the result: "Unit 1 complete: endpoint accepts queries"

### 5. VERIFICATION & QUALITY
After each unit:
- Check for build errors (hooks run automatically)
- Check for linting (hooks run automatically)
- Run tests (hooks run automatically)
- If red/broken: debug immediately using `/debug` skill
- If green: move to next unit

**Provocation Check (Phase 2b)**
Before writing a unit, ask:
- "Is there a simpler way to do this?"
- "What could go wrong here?"
- "Are we overcomplicating this?"

Document the answer in the commit message.

### 6. OPTIONAL: SUBAGENT REVIEW
If available and appropriate:
- Tag `@code-reviewer` for complex logic
- Tag `@db-reviewer` for schema/queries
- Cite their feedback in commits

### 7. ITERATION
If plan changes mid-build:
- Update the plan in chat
- Get user approval for new units
- Commit the delta
- Continue

### 8. WRAP UP
When all units are complete:
- Verify all tests pass
- Run any final integration tests
- Suggest `/wrap-up` to capture lessons and commit session

## Anti-Patterns
- ❌ Skip the plan — you'll build wrong things
- ❌ Modify tests mid-build — tests define the contract
- ❌ Leave broken code mid-stream — fix it before moving on
- ❌ Assume stack without asking — always check available skills
- ❌ Burn context on random tries — debug systematically using `/debug`

## Integration with Other Skills
- **`/tdd`**: Use when building features with high test coverage (recommended)
- **`/debug`**: Use when something breaks mid-build
- **`/learn`**: Use to capture gotchas and lessons
- **Stack-specific skills**: Use instead of implementing directly (e.g., `/new-component` for React)

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
