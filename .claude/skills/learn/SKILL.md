---
name: "learn"
description: "Capture a lesson and save it to CLAUDE.md with deduplication. Build the knowledge base."
---

# /learn — Lesson Capture

Use this skill to save lessons learned during development. Builds the knowledge base for this project.

## When to Use /learn
✅ **Do save these lessons:**
- A non-obvious gotcha you just discovered
- A trap that wasted time and should be documented
- A decision made and the reasoning
- A pattern that worked well
- A bug that was hard to diagnose
- A performance optimization you found
- A limitation you hit

❌ **Don't save these:**
- Generic knowledge (e.g., "always use prepared statements")
- Things obvious from the code
- One-off errors that won't repeat
- Stack tutorial content (use comments in code instead)

## Workflow

### 1. IDENTIFY THE LESSON
Ask yourself:
- What did I just learn?
- Would this be useful to remember next session?
- Is this specific to THIS project?
- Did this take time to figure out?

Examples of good lessons:
- "Database queries return NULL for missing rows; remember to handle this"
- "The API changed in v2.0; use endpoint X instead of Y"
- "Performance bottleneck: pagination is faster than sorting in-memory"
- "Don't use synchronous I/O here; it blocks the event loop"

Examples of bad lessons:
- "Use comments in code" (too generic)
- "Arrays start at 0" (too obvious)
- "Server crashed today" (too specific, won't repeat)

### 2. DEDUPLICATION: SEARCH EXISTING LESSONS
Before writing, search for similar lessons in CLAUDE.md files:

```bash
grep -r "keyword1" .claude/
grep -r "keyword2" .claude/
```

If you find a similar lesson:
- Check if it covers what you want to save
- If yes: don't add a duplicate, just reference it in session notes
- If no: consider if your lesson adds value or just repeats
- If it truly adds value: save it separately (both lessons might be useful)

### 3. IDENTIFY THE RIGHT CLAUDE.MD
Lessons are saved in the nearest CLAUDE.md to where they apply:

| Lesson Type | Save To |
|------------|----------|
| General project patterns | `PROJECT_ROOT/CLAUDE.md` |
| Database patterns | `src/db/CLAUDE.md` |
| API patterns | `src/api/CLAUDE.md` |
| UI patterns | `src/ui/CLAUDE.md` |
| Build/test patterns | `.claude/CLAUDE.md` |
| Framework-specific | Stack-specific directory |

Create the file if it doesn't exist.

### 4. WRITE THE LESSON
Format:
```
**[Short title]**: [What to do / not do]. [Why — what breaks otherwise.]
```

Keep it concise (2–3 sentences max).

Examples:

```
**Null check database results**: Remember to check if query returns
NULL or empty. Don't assume a row exists. Why: query returns NULL
for missing rows; if you try to access .name on NULL, you'll get
a null pointer error.
```

```
**Pagination before sorting**: Implement pagination at the query
level, not by sorting all results in memory. Why: sorting 10k rows
in-memory is 100x slower than using LIMIT in SQL.
```

```
**Event listeners in React useEffect**: Attach event listeners
inside useEffect with cleanup. Why: without cleanup, listeners
accumulate and fire multiple times.
```

```
**API v2 removed /users/all**: Use /users?limit=1000 instead.
Why: v2 removes the all endpoint for performance.
```

### 5. ADD CATEGORY COMMENT (OPTIONAL)
Use a comment tag to categorize:
```
<!-- DATABASE -->
**Null check database results**: ...

<!-- PERFORMANCE -->
**Pagination before sorting**: ...

<!-- REACT -->
**Event listeners in useEffect**: ...

<!-- API -->
**API v2 removed /users/all**: ...
```

### 6. PREVENT CONFLICTS
Before saving:
- Check the lesson doesn't contradict existing lessons
- If it does, discuss with user: "Existing lesson says X, but I want to save Y. Which is correct?"
- Only save after resolving conflict

### 7. SAVE IT
Append the lesson to the appropriate CLAUDE.md:

Example file structure:
```markdown
# CLAUDE.md for src/db/

## Lessons Learned

<!-- DATABASE -->
**Null check database results**: ...

<!-- DATABASE -->
**Pagination before sorting**: ...

<!-- PERFORMANCE -->
**Connection pooling**: ...
```

After saving, commit:
```bash
git add .claude/
git commit -m "LEARN: [lesson title]"
```

## Deduplication Details

### What is a duplicate?
- Same lesson, same wording (obviously duplicate)
- Same lesson, different wording (still duplicate, merge them)
- Related lessons (might both be useful, keep both)

Example:
```
Existing: "Always check for NULL after database queries"
New: "Database queries return NULL for missing rows; check before use"
→ These are the same lesson. Don't add new one.

Existing: "Database queries return NULL for missing rows"
New: "Use .isNone() helper to check for NULL values safely"
→ Related but different. Both are useful. Save both.
```

### How to deduplicate
1. Search for keywords in existing CLAUDE.md files
2. If you find similar lessons, read them carefully
3. If the exact same lesson exists: skip it
4. If the lesson is slightly different: decide if it adds value
5. If it's genuinely new information: save it

## Example Session

```
=== LESSON 1 ===
User: "I just found a bug. API endpoint doesn't handle special characters in names."
SKILL: "I'll save this lesson."
SKILL: [searches existing lessons]
SKILL: [finds no similar lesson]
SKILL: [creates lesson]
Lesson: "**API endpoint: URL-encode special characters**: When searching for names with
special characters (e.g., O'Neill), URL-encode them first. Why: unencoded special
chars break the API query parser."
SKILL: [appends to src/api/CLAUDE.md]
SKILL: [commits] "LEARN: URL encode special characters in API queries"

=== LESSON 2 ===
User: "Performance improved after paginating queries. We were loading too much data."
SKILL: "I'll save this lesson."
SKILL: [searches existing lessons]
SKILL: [finds: "Pagination before sorting: Implement pagination at query level"]
SKILL: "I found a similar lesson already saved. This reinforces that lesson but doesn't
add new information. Skipping duplicate. (Existing lesson already covers this.)"
SKILL: [in session notes] "Confirmed: pagination at query level is faster"

=== LESSON 3 ===
User: "Found the bug causing tests to fail. State wasn't reset between tests."
SKILL: "I'll save this lesson."
SKILL: [searches existing lessons]
SKILL: [finds: "Test fixtures: always reset state between tests"]
SKILL: "Similar lesson exists. But my version is more specific: 'beforeEach vs
beforeAll - use beforeEach to reset state, not beforeAll which shares state
between tests.' This adds value. Saving as new lesson."
SKILL: [appends to .claude/CLAUDE.md]
Lesson: "**beforeEach vs beforeAll**: Use beforeEach to reset state between tests,
not beforeAll. Why: beforeAll shares state between tests (causes flakes);
beforeEach resets state so each test is independent."
SKILL: [commits] "LEARN: use beforeEach for test setup, not beforeAll"
```

## Lesson Format Reference

```markdown
<!-- CATEGORY -->
**[Short title]**: [Action/principle]. [Why it matters / what breaks without it.]

Examples:

<!-- DATABASE -->
**Connection pooling**: Use connection pooling instead of creating new connections per query. Why: creating new connections is 10x slower.

<!-- PERFORMANCE -->
**Index frequently-queried columns**: Add database indexes to columns you query by (e.g., user.id, user.email). Why: without indexes, queries scan all rows.

<!-- API -->
**Rate limit API calls**: Implement rate limiting on API endpoints (e.g., 100 requests/minute). Why: without it, a single client can overload the server.

<!-- TESTING -->
**Mock external services**: Mock API calls in tests, don't hit real APIs. Why: real APIs are slow and flaky; tests should be fast and deterministic.

<!-- FRAMEWORK -->
**Use dependency injection**: Pass dependencies as parameters, don't use global singletons. Why: global state makes testing hard and causes hidden bugs.
```

## Integration with Other Skills
- **/dev**: After building features, save lessons via /learn
- **/debug**: After fixing bugs, save lessons via /learn (what you learned debugging)
- **/wrap-up**: Session ends by calling /learn for 1–3 lessons from the session

## Checklist
- [ ] Identified a lesson worth saving
- [ ] Searched for similar lessons (dedup)
- [ ] Identified the right CLAUDE.md file
- [ ] Wrote the lesson concisely (2–3 sentences)
- [ ] Added category comment (optional but recommended)
- [ ] Checked for conflicts with existing lessons
- [ ] Saved the lesson
- [ ] Committed with message

## Key Principles
1. **Specific to this project**: Don't save generic knowledge
2. **Worth remembering**: Would this help future you?
3. **Deduplication matters**: Don't repeat lessons
4. **Organized by location**: Save in nearest CLAUDE.md
5. **Concise**: 2–3 sentences max
6. **Why it matters**: Always explain why the lesson is important
