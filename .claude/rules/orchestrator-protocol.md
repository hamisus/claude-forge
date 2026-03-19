---
globs: ["**/*"]
---

# Orchestrator Protocol

When working on any non-trivial task, follow this contractor-mode loop:

## The Loop

```
1. PLAN    → What needs to happen? What files? What order?
2. BUILD   → Implement one unit at a time. Use skills when available.
3. VERIFY  → Does it compile/lint? Do tests pass?
4. REVIEW  → Check architecture compliance.
5. FIX     → Address issues (critical → major → minor).
6. SCORE   → Quality check: would this pass code review?
7. REPORT  → Tell the user what was done + what's next.
```

## Rules

- Never skip PLAN. Even for "quick fixes", state what you're about to do.
- Never skip VERIFY. Run the project's linter and tests after every logical unit.
- If VERIFY fails, go back to BUILD. Don't proceed with broken code.
- If you're unsure about a design decision, ASK the user. Don't guess.
- If a task touches more than 5 files, break it into smaller sub-tasks.
- Use skills instead of writing boilerplate from scratch.
- Delegate verbose operations to subagents to keep context clean.

## Guard Phrases

- Before planning: "Let me plan this before implementing."
- Before building: "Plan approved. Starting implementation."
- After each unit: "Unit complete. Verifying..."
- When stuck: "I'm going to investigate this systematically."
- When done: "Implementation complete. Ready for /wrap-up?"
