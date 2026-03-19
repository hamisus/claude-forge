---
name: "debug"
description: "Structured debugging guide — diagnose systematically, don't guess randomly. Stack-agnostic methodology."
---

# /debug — Structured Debugging

Use this skill when something is broken and you need to fix it. Prevents random trial-and-error that burns context.

## Core Principle
> **Read the FIRST error. Downstream errors are usually caused by the first one.**

Don't fix the 5th error if the 1st error is the root cause. Fix the first one, re-run, and see what happens.

## Workflow

### 1. CLASSIFY THE PROBLEM

Identify which category your problem falls into:

| Category | Symptoms | Examples |
|----------|----------|----------|
| **Build Error** | Code doesn't compile/parse | Syntax error, import not found, type mismatch |
| **Test Failure** | Tests run but fail | Assertion failed, unexpected output, timeout |
| **Runtime Error** | Code runs but crashes | Null pointer, division by zero, stack overflow |
| **Data Issue** | Wrong output, unexpected values | Query returns wrong results, cache is stale |
| **Network Issue** | Communication fails | Timeout, connection refused, DNS error |
| **UI Issue** | Visual or interaction problem | Button doesn't respond, content missing, layout broken |
| **Code Gen Issue** | Generated code is wrong | Scaffold created wrong structure, migration failed |

### 2. DIAGNOSE BY CATEGORY

#### BUILD ERROR
```
Step 1: Read the error message
  - Line number and file
  - What is the error (syntax, import, type)?

Step 2: Check the first error only
  - If multiple errors: fix the first one, re-run

Step 3: Common causes:
  - Syntax error: missing bracket, semicolon, quote
  - Import error: file doesn't exist, wrong path
  - Type error: wrong argument type, missing field
  - Dependency error: library not installed

Step 4: Fix and re-run build
```

#### TEST FAILURE
```
Step 1: Read the test output
  - Which test failed?
  - What was expected vs actual?

Step 2: Check the failing assertion
  - Example: "Expected [1, 2, 3], got [1, 2]"

Step 3: Common causes:
  - Implementation doesn't match test contract
  - Edge case not handled (empty input, special char, etc)
  - State not initialized correctly
  - Async issue (test runs before async completes)

Step 4: Fix implementation (don't change test)
Step 5: Re-run test

Key: DO NOT modify the test to make it pass. Tests define the contract.
```

#### RUNTIME ERROR
```
Step 1: Read the stack trace (first line is most important)
  - Which function crashed?
  - What was it trying to do?

Step 2: Identify the root cause
  - Null/undefined value?
  - Wrong type?
  - Resource not available?

Step 3: Common causes:
  - Null pointer: check for null before using
  - Array out of bounds: check length before accessing
  - Resource not open: check connection/file exists
  - Permission denied: check user permissions

Step 4: Add defensive check or fix the caller
Step 5: Re-run

Example:
  Error: Cannot read property 'name' of undefined
  Cause: Object is null/undefined
  Fix: Check if object exists before accessing property
```

#### DATA ISSUE
```
Step 1: What is the wrong output?
  - Example: query returns 5 results, expected 3

Step 2: Trace the data backwards
  - Where does the data come from? (database, API, cache?)
  - Is the source data correct?
  - Is the transformation correct?

Step 3: Add logging to trace data
  - Log at each step: input → transform → output
  - Example: log query, log results, log filter applied

Step 4: Common causes:
  - Wrong query: database search is too broad
  - Cache is stale: cache not invalidated after update
  - Transform is wrong: filter logic is inverted
  - Parser error: string parsed incorrectly

Step 5: Fix the root cause (query, cache, transform, parser)
```

#### NETWORK ISSUE
```
Step 1: What is the network error?
  - Timeout: request takes too long
  - Connection refused: server not running
  - DNS error: hostname not resolvable
  - 4xx error: client error (bad request, not found)
  - 5xx error: server error (internal server error)

Step 2: Check the basics first
  - Is the server running?
  - Is the URL correct?
  - Is the network connection available?
  - Is firewall blocking?

Step 3: Common causes:
  - Server not running: start it
  - Wrong URL: check hostname/port
  - Request malformed: check headers, body
  - Server error: check server logs
  - Timeout: server too slow or network too slow

Step 4: Check network request details
  - URL and method (GET, POST, etc)
  - Headers and auth tokens
  - Request body
  - Response status and body

Step 5: Fix and retry
```

#### UI ISSUE
```
Step 1: What is the problem?
  - Button doesn't respond: click handler not firing
  - Content missing: element not rendered
  - Layout broken: CSS not applied
  - Page loads but blank: JS error or API failure

Step 2: Check browser console
  - Any JavaScript errors?
  - Any CSS warnings?
  - Any network errors (failed API calls)?

Step 3: Common causes:
  - Element selector wrong: can't find element to attach handler
  - Event handler not registered: click handler not attached
  - CSS not loaded: stylesheet path wrong
  - API call failed: content can't load
  - Async not awaited: element renders before data arrives

Step 4: Check the DOM
  - Is the element in the page?
  - Is it hidden (display: none, visibility: hidden)?
  - Is it behind another element?

Step 5: Fix and reload
```

#### CODE GEN ISSUE
```
Step 1: What was generated wrong?
  - Wrong structure: scaffold created wrong files
  - Wrong names: variables/functions named incorrectly
  - Missing files: some files not created
  - Wrong content: file has wrong boilerplate

Step 2: Check the code gen template
  - Is the template correct?
  - Are the variables substituted correctly?

Step 3: Common causes:
  - Template has a typo
  - Variable not provided to template
  - Template uses wrong field name
  - Caching: old template was used

Step 4: Re-run code gen with correct inputs
Step 5: Or fix the generated files manually
```

### 3. ANTI-PATTERN: RANDOM TRIES
❌ DON'T do this:
```
- Try changing the condition from < to <=
- Try adding .toString()
- Try wrapping in a try-catch
- Try restarting the server
- Try clearing cache
- ...repeat until something works
```

This burns context and you learn nothing.

✅ DO this instead:
```
- Read the error message carefully
- Understand the root cause
- Make ONE targeted fix
- Verify it works
- Commit the fix
- Save the lesson via /learn
```

### 4. AFTER FIXING: SAVE THE LESSON
Once fixed, use `/learn` to capture:
- **What went wrong**: specific symptom
- **Root cause**: why it happened
- **How to fix**: step-by-step fix
- **Why it's non-obvious**: what makes this tricky to debug

Example lesson:
```
**Test failure with async: Forgot to await Promise**

When tests fail with "expected X, got undefined", check if the
code is awaiting async calls. Promises that aren't awaited return
undefined immediately. Use `await` or `.then()` to wait for result.

Why non-obvious: The error message doesn't mention async or Promise.
```

## Debugging Workflow Summary

1. **Classify**: Build error? Test failure? Runtime error? Data issue? Network issue? UI issue? Code gen issue?
2. **Read the first error**: Don't chase downstream errors
3. **Diagnose**: Use the category-specific steps above
4. **Fix**: Make one targeted fix
5. **Verify**: Re-run to confirm it works
6. **Learn**: Use `/learn` to save the lesson
7. **Commit**: Commit the fix with a clear message

## Example: Debugging a Test Failure

```
=== PROBLEM ===
Test output:
  ✗ search returns results for "alice"
    Expected: [User(id=1, name="alice")]
    Got: []

=== CLASSIFICATION ===
This is a TEST FAILURE

=== DIAGNOSIS ===
The test expects search to return alice, but got empty list.

Step 1: Is the test correct?
  - Yes, test code looks right

Step 2: Does alice exist in test data?
  - Need to check database/fixture setup

Step 3: Is the search query correct?
  - Check if search is using the right field

Step 4: Trace data:
  - Query string: "alice" ✓
  - Database called? (add log) → Yes, query runs
  - Database returns results? (add log) → NO! Query returns []
  - Why? (check database) → alice not in test data!

=== ROOT CAUSE ===
Test fixture doesn't create alice user before running search test.

=== FIX ===
Add alice to test data fixture:
  beforeEach(() => {
    db.insert({name: "alice", id: 1})
  })

=== VERIFY ===
Re-run test → ✓ search returns results for "alice"

=== LEARN ===
Save lesson: "Test fixture setup: must insert test data before querying"

=== COMMIT ===
git commit -m "FIX: add test fixture for alice user"
```

## Example: Debugging a Runtime Error

```
=== PROBLEM ===
Error: Cannot read property 'name' of undefined
  at getUserName (users.ts:42)
  at renderUser (render.ts:15)

=== CLASSIFICATION ===
This is a RUNTIME ERROR (null pointer)

=== DIAGNOSIS ===
Something is trying to read .name from an undefined object.

Stack trace shows: getUserName calls renderUser at line 15
Let me check line 42 in users.ts

Code: return user.name
Problem: user is undefined

Where does user come from?
  return user from database query

Is database returning undefined?
  Check: if (user === undefined) → yes!

Why is database returning undefined?
  Check query: SELECT * FROM users WHERE id = ?
  Is the query parameter passed correctly?
  Check: id parameter is NaN!

Why is id NaN?
  Check caller: renderUser receives user ID as string
  But code tries to use it as number without converting

=== FIX ===
Convert id to number:
  const userId = parseInt(id, 10)
  const user = database.query(userId)

=== VERIFY ===
Re-run → no error, function works

=== LEARN ===
Save lesson: "Type mismatch: string ID from URL/API must be parsed to number before database query"

=== COMMIT ===
git commit -m "FIX: parse user ID from string to number"
```

## Checklist
- [ ] Classified the problem type
- [ ] Read the first/root error
- [ ] Diagnosed using category-specific steps
- [ ] Made one targeted fix
- [ ] Verified the fix works
- [ ] Saved the lesson via /learn
- [ ] Committed with clear message

## Key Principles
1. **Root cause first**: Fix the root, not symptoms
2. **Read errors carefully**: The error message has the answer
3. **Trace backwards**: Follow data from output back to source
4. **One fix at a time**: Avoid random trial-and-error
5. **Verify after each fix**: Make sure fix actually works
6. **Save lessons**: Next time you see this, you'll know
