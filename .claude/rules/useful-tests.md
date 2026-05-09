# Rule: Useful Tests

> Single source of truth for "what makes a test worth writing".
> Referenced by `/dev`, `/tdd`, `/phase-review`, `/kanban-prep`.

## Definition

**A useful test verifies an externally observable behavior that a real user, caller, or contract would care about.**

If a test passes when the implementation is broken in plausible ways, it is not useful — regardless of what the coverage report says.

---

## What counts (✅)

- **Drives code through a public entry point** — HTTP route, exported function, CLI command, rendered component, message handler.
- **Asserts on output the consumer would actually see** — response body, return value, rendered DOM, persisted DB row, emitted event.
- **Fails when behavior breaks** — change a `+` to a `-` in the implementation; at least one test should go red.
- **Pins contract boundaries** — request/response shape, error codes, status codes, side effects, persisted state.
- **At least one end-to-end test per feature** — a test that exercises a realistic user flow from entry to outcome (HTTP request → response, click → updated UI, command → file written).
- **UI tests assert "given props/route, the user sees X and can do Y"** — Playwright/RTL/widget-test style. Visible content, interactive behavior.
- **Edge cases from `PROJECT_SPEC.md`** — every documented edge case has a test that exercises it.

---

## What does NOT count (❌)

- **"Function exists" / "imports without error" / "constructor doesn't throw"** — duplicates the loader's job.
- **Mocks of the system under test** — mocking the thing you're testing makes the test tautological.
- **`expect(spy).toHaveBeenCalledOnce()` with no behavioral assertion** — call counts without consequence checks tell you nothing about correctness.
- **Tests that duplicate the type-checker's job** — TypeScript/mypy already verify shapes; don't write runtime tests that re-assert types.
- **Tautologies** — `expect(true).toBe(true)`, `expect(x).toBe(x)`, snapshot of an empty render.
- **Assertion-less tests** — test body has no `expect` / `assert` / equivalent.
- **Snapshot-only tests on non-deterministic output** — they pin noise, not behavior. Snapshots are OK for stable, intentional structure (e.g., serializer output).
- **`it("works")` / `test("happy path")` with no specifics** — vague titles signal vague tests.

---

## The audit (runs after GREEN, before commit)

Three steps. The first two are mechanical; the third is judgment.

### Step 1 — Mechanical scan

Grep new/changed test files for smell patterns:
- Test bodies with no `expect` / `assert` / `should` / `must`
- Tests whose only assertion is `toHaveBeenCalled` (any variant) with no follow-up behavioral check
- Snapshot-only tests (`toMatchSnapshot` / `toMatchInlineSnapshot` as the sole assertion)
- Mocks of the file/module under test (the SUT mocks itself)
- Test titles matching `/^(it|test)\((['"`])(works|ok|test|happy path)\2/`

Output: list of suspect tests with file:line.

### Step 2 — Coverage map

For each new or changed source unit (function/class/component), list which test references it (by import or by entry-point invocation). Flag any unit with **zero** referencing tests.

This is *referential* coverage — does any test exercise this thing? — not line coverage.

### Step 3 — Judgment pass (mandatory, LLM-driven)

For each new test, the agent must answer:

> If I broke the implementation in plausible ways — wrong return type, off-by-one, swapped branches, returned cached/stale data, threw instead of returning, returned `null` instead of `undefined`, returned the wrong key — would this test fail?

Answer format (in commit message body, under `useful-tests-audit:`):

```
useful-tests-audit:
- tests/users.test.ts::"creates a user" — would catch: wrong response shape, missing id field, wrong status. Would NOT catch: persisted-vs-in-memory difference (no DB read-back).
- tests/auth.test.ts::"rejects expired token" — would catch: missing 401, wrong error code. Would NOT catch: token leakage in logs.
- ...
```

**At least one mutation per test must be named.** "Would catch all bugs" is not an acceptable answer.

If a test answers "would not catch the obvious mutation" → strengthen the test, or log an override with reason.

---

## Override path

If you genuinely cannot write a useful test for a unit (rare — usually means the unit is wrong, not the test), log an override in the commit footer:

```
useful-tests-skip-reason: <one-line reason>
```

Phase-review surfaces the skip-rate. >20% triggers a discussion.

---

## How phase-review evaluates this

The "Test Quality" dimension of the 5-dimension phase-review score (0–20):

| Score | What it means |
|------:|---------------|
|     0 | Many tautologies; no e2e for any feature; audit blocks ignored. |
|    10 | Mixed — some useful tests, some smells. |
|    16 | Every shipped feature has ≥1 useful end-to-end test + per-unit useful tests. |
|    20 | Above + every documented edge case from `PROJECT_SPEC.md` has a test. |

---

## Honest limitations

Step 3 is **judgment-augmented-by-process**, not pure automation.

The rigorous alternative is **mutation testing** (Stryker for JS/TS, mutmut for Python, mutest for Rust, etc.) — actually mutate the source and verify a test fails. It's slower, stack-specific, and not always practical per-commit. Per-phase mutation runs are a reasonable upgrade path; per-commit "thought-experiment mutation" via the judgment pass is the floor.

The rubric here pins the criteria. Commit messages capture the answers. Phase-review re-checks. The agent that runs the audit can rubber-stamp; the rubric is designed to make rubber-stamping visible (vague answers in commit messages are themselves a smell).

---

## Quick examples

**Useful** (a test that earns its keep):
```ts
test("POST /api/users with duplicate email returns 409", async () => {
  await db.users.create({ email: "x@y.com", name: "first" });
  const res = await request(app).post("/api/users").send({ email: "x@y.com", name: "second" });
  expect(res.status).toBe(409);
  expect(res.body.code).toBe("DUPLICATE_EMAIL");
  expect(await db.users.count({ email: "x@y.com" })).toBe(1); // no second insert
});
```
Catches: wrong status code, missing error code, accidental second insert, swallowed unique constraint.

**Not useful** (a test that adds noise):
```ts
test("createUser works", () => {
  const createUser = jest.fn().mockReturnValue({ id: 1 });
  const result = createUser({ email: "x@y.com" });
  expect(createUser).toHaveBeenCalled();
  expect(result.id).toBe(1);
});
```
Catches: nothing. The test mocks the SUT and asserts on the mock's own return value.
