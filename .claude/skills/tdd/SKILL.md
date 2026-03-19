---
name: "tdd"
description: "Test-driven development workflow — RED → GREEN → REFACTOR cycle. Stack-agnostic pattern."
---

# /tdd — Test-Driven Development

Use this skill when you want to build features with test-driven development (TDD).

## When to Use TDD
- ✅ Core business logic (benefits from having a contract)
- ✅ APIs and data layers (easy to test, important to verify)
- ✅ Libraries and utilities (reused across codebase)
- ✅ Complex algorithms (tests verify correctness)
- ❌ UI components (often harder to test; see stack-specific UI skills)
- ❌ Configuration (usually not worth testing)
- ❌ One-off scripts (can skip TDD)

**Rule of thumb**: If you're building something that will be reused, tested by others, or has complex logic, use TDD.

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

❌ **Write tests that are too loose**: Tests should catch real bugs
- ❌ Bad: "test that search returns something"
- ✅ Good: "test that search returns specific users for specific query"

❌ **Refactor without running tests**: Always verify tests still pass after refactoring
- If you change code and tests fail, you've changed behavior.

❌ **Add new features during refactoring**: Refactoring is only for improving existing code
- If you want a new feature, write new tests first (go back to RED phase).

## Integration with /dev Skill
- Use `/dev` for overall feature workflow
- Use `/tdd` within `/dev` for specific features
- Example workflow: `/dev` calls `/tdd` for the data layer, then stack-specific skills for UI

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
