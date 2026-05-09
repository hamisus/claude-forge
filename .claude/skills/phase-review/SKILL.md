---
name: "phase-review"
description: "Phase end audit — capture lessons, detect tech debt, score quality. Go/no-go for next phase."
---

# /phase-review — Phase End Audit

Run this skill at the end of each phase to audit work, detect tech debt, and decide if you're ready for the next phase.

## Workflow

### 1. READ THE COMPLETED PHASE
From `PHASES.md`, identify:
- Which phase just completed? (e.g., Phase 2b)
- Which tasks were planned?
- Which tasks were completed?
- Which tasks remain?

### 2. SCAN FOR LESSONS
Review work done in this phase:
- Scan commits for gotchas or non-obvious fixes
- Ask user: "Any lessons from this phase we should document?"
- Use `/learn` to save each lesson

Example prompt to user:
```
Phase 2b complete! Quick lessons check:
- Did anything surprise you or take longer than expected?
- Any decisions made that should be documented?
- Any workarounds or "remember this" moments?
```

### 3. DETECT TECH DEBT
Scan codebase for signs of technical debt:

#### A. TODO/FIXME Comments
```bash
grep -r "TODO\|FIXME\|HACK\|XXX" src/
```

For each comment:
- What is it? (e.g., "TODO: add validation")
- Is it important? (should be fixed soon vs. can wait)
- Who should fix it? (current phase or later)

Capture significant tech debt items in `CLAUDE.md`:
```markdown
## Tech Debt
- [ ] Add input validation to search endpoint (Phase 2c)
- [ ] Optimize database queries (Phase 3a)
- [ ] Refactor authentication module (defer to Phase 4)
```

#### B. Large Files (>300 lines)
```bash
find src/ -name "*.js" -o -name "*.py" -o -name "*.ts" | \
  xargs wc -l | sort -rn | head -20
```

For each file >300 lines:
- Can it be split into smaller files?
- Is it a legitimate large module or code smell?
- Should it be refactored in next phase?

#### C. Code Duplication
Look for repeated patterns:
- Similar functions that could be extracted
- Copy-paste code
- Repeated logic across modules

#### D. Unused Code
Look for:
- Unused imports
- Dead code paths
- Old comments or commented-out code

#### E. Test Quality
Audit per `.claude/rules/useful-tests.md` — line coverage % is **not** the primary metric.

Check:
- Does every shipped feature have ≥1 useful end-to-end test (drives a public entry point, asserts on observable output)?
- Does every documented edge case from `PROJECT_SPEC.md` have a test?
- Do new source units each have ≥1 referencing test (per the Coverage Map step)?
- Does every commit on this phase's branch include a `useful-tests-audit:` block in its body? (`git log --grep="useful-tests-audit:"` — count vs. total commit count.)
- Smell scan: any tautologies, snapshot-only tests, mocks-of-SUT, `toHaveBeenCalled`-only tests, vague `it("works")` titles?
- TDD-skip rate: count `tdd-skip-reason:` entries on the phase's commits. If >20%, surface for discussion.
- Useful-tests-skip rate: count `useful-tests-skip-reason:` entries. If >5%, the gate is being bypassed too often — investigate.

Goal: tests that would catch plausible mutations of the implementation, not tests that pad coverage numbers.

### 4. AUDIT CLAUDE.MD FILES
Review all CLAUDE.md files for accuracy:

- Are lessons still relevant?
- Are they accurate given what was built?
- Are there contradictions?
- Should anything be updated or removed?

### 5. UPDATE ROOT CLAUDE.MD
Update `PROJECT_ROOT/CLAUDE.md`:

```markdown
# Project Overview

## Current Status
- Phase: 2b (complete)
- Started: 2026-03-15
- Ended: 2026-03-19
- Focus: User search API

## What Was Built
- [✓] GET /api/users/search endpoint
- [✓] Pagination support
- [✓] Integration tests
- [✓] Database queries optimized

## What Remains
- [ ] Search filters UI (Phase 2c)
- [ ] Rate limiting (Phase 3a)
- [ ] Analytics logging (Phase 3b)

## Tech Debt Detected
- TODO: validate search query length (src/api/users.ts:42)
- Large file: src/db/queries.ts (456 lines) - consider splitting
- No tests for error cases (e.g., database connection failure)

## Phase 2b Lessons
- Database pagination is faster than sorting in-memory
- Always URL-encode special characters in API queries
- BeforeEach resets state; beforeAll doesn't (use beforeEach)

## Key Decisions Made
- Decided AGAINST caching (not needed yet)
- Chose pagination over cursor-based nav (simpler)
- Implemented soft deletes for audit trail

## Next Phase Goals
- Build search UI with filters
- Implement rate limiting
- Add analytics logging
```

### 6. UPDATE PHASES.MD
Update the phase section in `PHASES.md`:

```markdown
## Phase 2b: Development ✓ COMPLETE
- [x] API endpoint
- [x] Pagination support
- [x] Integration tests
- [x] Database queries
- [x] Handoff documentation

Status: COMPLETE
Quality score: 84/100
Tech debt: low (1 TODO, no major issues)
Ready for next phase: YES
```

### 7. QUALITY GATE SCORE
Score the phase on 5 dimensions (each 0–20, max 100):

| Dimension | Questions | Score |
|-----------|-----------|-------|
| **Completeness** | All planned tasks done? Any scope creep? | /20 |
| **Code Quality** | Is code clean, readable, maintainable? | /20 |
| **Test Quality** | Useful tests per `.claude/rules/useful-tests.md`? E2E per feature? Edge cases covered? Audit blocks present? | /20 |
| **Design/UX** | Does feature work well? Intuitive? | /20 |
| **Documentation** | Is work documented? Lessons captured? | /20 |

General scoring guide:
- 16–20: Excellent, no concerns
- 11–15: Good, minor issues
- 6–10: Acceptable, some concerns
- 0–5: Poor, significant issues

**Test Quality rubric** (specifically):
| Score | What it means |
|------:|---------------|
|   0–5 | Many tautologies; no e2e for any feature; audit blocks missing or rubber-stamped. |
|  6–10 | Mixed — some useful tests, some smells. Skip-rate >20%. |
| 11–15 | Most features have useful tests, but e2e coverage is patchy or some edge cases untested. |
|    16 | Every shipped feature has ≥1 useful end-to-end test + per-unit useful tests. Audit blocks present and concrete. |
|    20 | All of the above + every documented edge case from `PROJECT_SPEC.md` has a test. TDD-skip and useful-tests-skip rates both <5%. |

Example scorecard:
```
Quality Gate Score: Phase 2b

Completeness: 18/20 (all tasks done, 1 minor scope creep)
Code Quality: 15/20 (clean code, but 1 file >300 lines)
Test Quality: 16/20 (e2e per feature, audit blocks present, but 2 documented edge cases untested)
Design/UX: 19/20 (API works well, excellent pagination)
Documentation: 15/20 (lessons captured, but CLAUDE.md could be more detailed)

TOTAL: 84/100

Status: PASS (>80)
Recommendation: Ready for next phase
```

### 8. QUALITY GATE DECISION

**If score ≥ 80:**
✅ PASS — Ready for next phase
- Document the score
- Suggest moving to next phase

**If score < 80:**
⚠️ FAIL — Fix issues before next phase
- Identify the lowest-scoring dimensions
- Suggest fixes for each:
  - Low completeness: finish remaining tasks
  - Low code quality: refactor large files
  - Low test quality: strengthen tests per `.claude/rules/useful-tests.md` (add e2e, cover edge cases, replace tautologies, ensure audit blocks present)
  - Low design/UX: iterate based on feedback
  - Low documentation: capture lessons and decisions

Example:
```
Quality Gate: 72/100 (FAIL — below 80 threshold)

Issues:
- Code Quality (12/20): src/api/users.ts is 450 lines, needs splitting
- Test Quality (10/20): no e2e tests; useful-tests-audit blocks rubber-stamped on 4 commits; 3 edge cases from PROJECT_SPEC.md untested

Recommendations:
1. Split src/api/users.ts into endpoint.ts + validation.ts + handlers.ts
2. Add an e2e test that drives the full user-search flow (HTTP request → DB read-back → response shape)
3. Cover the 3 untested edge cases (empty query, sql-injection-shaped query, pagination beyond last page)
4. Re-do the useful-tests-audit on the 4 rubber-stamped commits with concrete mutation answers

Estimated effort: 4 hours
Suggest: Fix these before moving to Phase 2c
```

### 9. RUN /skill-check
Meta-skill audit. Call `/skill-check` to verify:
- All skills are up-to-date
- No stale or incomplete skills
- New skills needed based on what was built

### 10. COMMIT DOC UPDATES
```bash
git add CLAUDE.md PHASES.md .claude/
git commit -m "[PHASE REVIEW] Phase 2b complete: quality 84/100, tech debt low, ready for Phase 2c"
```

### 11. PRESENT SUMMARY
Show user a summary card:

```
╔════════════════════════════════════════════════╗
║              PHASE 2b REVIEW                   ║
╚════════════════════════════════════════════════╝

WORK COMPLETED
✓ API endpoint (GET /api/users/search)
✓ Pagination support
✓ Integration tests (8 tests, all passing)
✓ Database queries optimized

LESSONS LEARNED
• Pagination faster than sorting in-memory
• URL-encode special characters in queries
• Use beforeEach for test setup, not beforeAll

TECH DEBT DETECTED
• 1 TODO comment (validate search query)
• 1 large file (src/db/queries.ts, 456 lines)
• Missing error case tests

QUALITY SCORE
╔─────────────┬────────┬────╗
║ Dimension   │ Score  │ OK ║
╠─────────────┼────────┼────╣
║ Complete    │ 18/20  │ ✓  ║
║ Code Quality│ 15/20  │ ✓  ║
║ Tests       │ 17/20  │ ✓  ║
║ Design      │ 19/20  │ ✓  ║
║ Docs        │ 15/20  │ ✓  ║
╠─────────────┼────────┼────╣
║ TOTAL       │ 84/100 │ ✓  ║
╚─────────────┴────────┴────╝

STATUS: PASS ✓
Ready for Phase 2c

NEXT PHASE GOALS
1. Build search UI with filters
2. Implement rate limiting
3. Add analytics logging
```

## When to Use /phase-review
- ✅ End of each phase
- ✅ Before starting next phase
- ✅ Quarterly project audits
- ✅ Before code reviews or release

## Integration with Other Skills
- **/learn**: Save lessons before phase-review
- **/skill-check**: Run to verify skills are up-to-date
- **/wrap-up**: End session with wrap-up, then phase-review at phase end

## Checklist
- [ ] Read completed phase from PHASES.md
- [ ] Scan for lessons and call /learn for each
- [ ] Detected tech debt (TODOs, large files, duplication)
- [ ] Audited CLAUDE.md files for accuracy
- [ ] Updated root CLAUDE.md with status and decisions
- [ ] Updated PHASES.md with completion status
- [ ] Calculated quality gate score (all 5 dimensions)
- [ ] Made go/no-go decision (≥80 vs <80)
- [ ] Ran /skill-check
- [ ] Committed updates
- [ ] Presented summary to user

## Key Principles
1. **Quality gates matter**: 80/100 is the pass threshold
2. **Tech debt is real**: Document it, don't ignore it
3. **Lessons stick**: Save every non-obvious finding
4. **Continuous improvement**: Each phase should improve
5. **Go/no-go**: Clear decision before next phase
6. **Transparency**: Show user the scorecard, explain scoring
