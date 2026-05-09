---
name: "tdd"
description: "Test-driven development workflow — RED → GREEN → REFACTOR cycle. Stack-agnostic pattern."
---

# /tdd — Test-Driven Development

**TDD is the default cycle for every unit built in `/dev`.** This skill describes how the cycle runs. To opt out for a specific unit, see "Override path" in `/dev` Step 4 — silent skips are forbidden.

## When TDD is hardest (and what to do anyway)

TDD applies everywhere; some places just need different test shapes:

- **UI components** — write a smoke render test (RTL/widget-test/Playwright) that asserts what the user sees and can do. *Harder, not exempt.*
- **Configuration / constants** — usually no behavior to test, so log `tdd-skip-reason: config-only edit` (and keep the change small enough that "config-only" is true).
- **Throwaway scripts** — log `tdd-skip-reason: throwaway exploration`. If the script becomes load-bearing, write tests then.
- **Migrations / schema changes** — write a migration test that runs up + down and asserts the resulting schema shape.
- **External-service integration** — write a contract test against a mock that mirrors the real service's documented response shapes (this is what "useful" means here — see `.claude/rules/useful-tests.md`).

**The rule isn't "test everything." The rule is: if there's behavior, a useful test exists for it. If there's no behavior, log the skip.**

## What makes a test useful

See `.claude/rules/useful-tests.md` for the full rubric. Short version:

- ✅ Drives code through a public entry point (HTTP route, exported function, rendered component, CLI command).
- ✅ Asserts on output a real consumer would see.
- ✅ Fails when the implementation breaks in plausible ways.
- ❌ Mocks the system under test.
- ❌ Asserts only call counts (`toHaveBeenCalled`) with no behavioral check.
- ❌ Tautologies, snapshot-only, assertion-less tests.

**RED-phase tests must be useful by this definition.** A failing test that wouldn't catch a real bug is wasted RED. Write the test as if a stranger had to maintain the implementation.

## The TDD Cycle

### PHASE 1: RED — Write Failing Tests

**Step 1: Define the contract**
- What should this code do?
- What are the inputs and outputs?
- What are the edge cases?
- What should it NOT do?

Example (stack-agnostic pseudocode):
```
Test: "search finds users by name"
  Input: query="alice"
  Expected: returns list of users matching "alice"

Test: "search returns empty list if no matches"
  Input: query="zzzz"
  Expected: returns []

Test: "search trims whitespace from query"
  Input: query="  bob  "
  Expected: same as query="bob"

Test: "search rejects queries < 2 chars"
  Input: query="a"
  Expected: throws error or returns []
```

**Step 2: Write the tests**
- Use your stack's testing framework (Jest, pytest, RSpec, Go testing, etc.)
- Focus on: happy path, edge cases, error cases
- Don't worry about implementation yet

**Step 3: Run tests**
- Tests should FAIL (you haven't implemented yet)
- Capture the failure output
- **COMMIT THIS STATE**: "RED: failing tests for search feature"
  - This is a checkpoint. You now have a contract to code against.

**Critical principle**: DO NOT MODIFY THE TESTS while implementing.
- Tests define the contract
- If tests need to change, discuss with user first

### PHASE 2: GREEN — Implement Until Tests Pass

**Step 1: Implement the minimal code to pass tests**
- Don't over-engineer
- Don't add features the tests don't ask for
- Your goal: make tests pass, nothing more

Example pseudocode:
```
function search(query):
  if query is empty:
    throw error
  if query length < 2:
    throw error

  query = query.trim()

  results = database.query("SELECT * FROM users WHERE name CONTAINS ?", query)
  return results
```

**Step 2: Run tests**
- Iterate: fix test failures one at a time
- After each fix: run tests
- When all tests pass: COMMIT
  - Message: "GREEN: search feature tests passing"

**Step 3: Verify all tests pass**
- Example: "8 passed, 0 failed"
- If any test still fails: don't move on, debug it now

**Key**: DO NOT MODIFY TESTS to make them pass. If a test fails, change the implementation, not the test.

### PHASE 3: REFACTOR — Improve Code While Keeping Tests Green

**Step 1: Improve the implementation**
- Extract functions
- Remove duplication
- Improve variable names
- Add comments for non-obvious logic
- Optimize performance (if needed)

Example refactorings:
- Extract `validate_query()` function
- Extract `normalize_query()` function
- Cache database connection
- Add logging

**Step 2: Run tests after each refactor**
- Tests should STILL PASS
- If a test fails: you broke something, revert this change

**Step 3: Commit each refactor**
- Message: "REFACTOR: extract validation logic into helper"
- Keep commits small and focused

**Key principle**: Tests stay GREEN throughout refactoring. If they turn RED, you've changed behavior — revert and try a different approach.

## Example: Complete TDD Cycle

```
=== RED PHASE ===
User: "Build a user search function"

SKILL: "I'll write tests first. Here's what search should do:
- Find users by name (case-insensitive)
- Support pagination
- Reject empty queries
- Handle special characters in queries"

SKILL: [writes tests in test file]
Test 1: search("alice") returns users with "alice" in name
Test 2: search("ALICE") returns same as search("alice")
Test 3: search("") throws error
Test 4: search("name") with page=2 returns next 10 results
Test 5: search("o'neil") handles apostrophe correctly

SKILL: [runs tests]
Output: "8 failed, 0 passed"
SKILL: [commits] "RED: user search tests (failing, as expected)"

=== GREEN PHASE ===
SKILL: "Now I'll implement the code to make tests pass."

SKILL: [implements search function]
SKILL: [runs tests]
Output: "5 failed, 3 passed"

SKILL: "Still some failures. Let me fix the pagination test..."
SKILL: [updates implementation]
SKILL: [runs tests]
Output: "8 passed, 0 failed"

SKILL: [commits] "GREEN: search function tests passing"

=== REFACTOR PHASE ===
SKILL: "Now I'll clean up the code."

SKILL: [extracts validation into helper function]
SKILL: [runs tests]
Output: "8 passed, 0 failed"
SKILL: [commits] "REFACTOR: extract query validation into helper"

SKILL: [extracts normalization into helper function]
SKILL: [runs tests]
Output: "8 passed, 0 failed"
SKILL: [commits] "REFACTOR: extract query normalization into helper"

SKILL: "TDD cycle complete! All tests passing, code is clean."
```

## Common Mistakes

❌ **Skip RED phase**: Don't just write code and then add tests
- Tests define the contract. Without tests first, you don't have a contract.

❌ **Modify tests to pass**: If tests fail, fix the code, not the tests
- This defeats the purpose of TDD.

❌ **Write tests that are too specific**: Tests should verify behavior, not implementation
- ❌ Bad: "test that function calls database.query() exactly once"
- ✅ Good: "test that function returns 10 results for this query"

❌ **Write tests that are too loose**: Tests should catch real bugs (per `.claude/rules/useful-tests.md`)
- ❌ Bad: "test that search returns something"
- ❌ Bad: "test that search calls db.query()" (call count without behavior check)
- ❌ Bad: snapshot-only test of search results
- ✅ Good: "test that search returns specific users for specific query, with stable order, and excludes soft-deleted rows"

❌ **Refactor without running tests**: Always verify tests still pass after refactoring
- If you change code and tests fail, you've changed behavior.

❌ **Add new features during refactoring**: Refactoring is only for improving existing code
- If you want a new feature, write new tests first (go back to RED phase).

## Useful Tests Verification

Before the final commit (after GREEN, before REFACTOR is fully landed), `/dev` runs the **Useful Tests Gate** — a three-step audit defined in `.claude/rules/useful-tests.md`:

1. **Mechanical scan** — grep for smell patterns.
2. **Coverage map** — every new source unit must have at least one test referencing it.
3. **Judgment pass** — for each new test, name one plausible mutation it would not catch. Document in commit body under `useful-tests-audit:`.

The gate fails if:
- A new test passes the mechanical scan only by coincidence (e.g., `expect(spy).toHaveBeenCalledOnce()` with no behavioral follow-up).
- A new source unit has zero referencing tests.
- A test admits it would not catch an obvious mutation.

When the gate fails: strengthen the test, or log `useful-tests-skip-reason:` in the commit footer (audit visible to phase-review; >20% rate triggers a discussion).

## Integration with /dev Skill

`/dev` defaults to `/tdd` for every unit. `/tdd` is no longer opt-in — it's the cycle the unit runs through unless a `tdd-skip-reason` is logged. Stack-specific implementation skills (e.g., `/new-component`) typically run inside `/tdd`'s GREEN phase, generating code to satisfy the failing tests written in RED.

## Integration with /debug Skill
- If tests fail and it's not obvious why, use `/debug` to diagnose
- `/debug` provides structured debugging methodology

## Stack-Agnostic Principle
This skill describes the TDD pattern, not specific commands:
- Don't assume the test framework (Jest, pytest, RSpec, Go testing, etc.)
- Don't assume the programming language
- The cycle (RED → GREEN → REFACTOR) applies everywhere

Stack-specific test runners will be available in stack-specific skills.

## Checklist
- [ ] Wrote tests for happy path
- [ ] Wrote tests for edge cases
- [ ] Wrote tests for error cases
- [ ] All tests fail initially (RED)
- [ ] Committed failing tests as checkpoint
- [ ] Implemented code to pass tests
- [ ] All tests pass (GREEN)
- [ ] Committed passing implementation
- [ ] Refactored code
- [ ] Tests still pass (GREEN)
- [ ] Committed refactorings
- [ ] Pushed to git

## Key Principles
1. **Red first**: Tests fail before code exists
2. **Green next**: Implement minimum code to pass tests
3. **Refactor last**: Improve code while keeping tests green
4. **Contract matters**: Tests define what the code should do
5. **Don't cheat**: Don't modify tests to make them pass
6. **Small cycles**: Do RED-GREEN-REFACTOR in tight loops
