# Claude Forge — Workflow Quick Reference

## Daily Development Session

```
1. Open Claude Code in your project
2. Claude reads Session Handoff from CLAUDE.md (automatic)
3. /dev <feature-to-build>
   → Context loaded, plan produced, you review
   → Build in small units, hooks verify each one
   → Tests run, architecture checked
4. /wrap-up
   → Lessons captured, code committed, handoff written
```

## Commands Cheat Sheet

| Command | When | What it does |
|---------|------|-------------|
| `/scope` | Project start | Interactive discovery conversation |
| `/spec` | After /scope | Generate spec + architecture docs |
| `/bootstrap` | After /spec | Generate custom skills, hooks, rules |
| `/dev <feature>` | Every session | Plan → build → verify loop |
| `/wrap-up` | End of session | Commit + lessons + handoff |
| `/tdd <feature>` | Test-first work | RED → GREEN → REFACTOR |
| `/debug <problem>` | When stuck | Structured diagnosis |
| `/learn` | After discovery | Capture a lesson |
| `/phase-review` | End of phase | Audit + quality score |
| `/skill-check` | Periodically | Audit skill accuracy |

## Agents

| Agent | When | What it does |
|-------|------|-------------|
| `@code-reviewer` | After building | Reviews architecture + quality |
| `@test-writer` | Need tests | Generates comprehensive tests |
| `@doc-keeper` | Periodically | Checks doc accuracy |

## Context Management

| Context % | Action |
|-----------|--------|
| 0-60% | Normal work |
| 60% | Consider `/wrap-up` + `/clear` |
| 80% | MUST `/wrap-up` + `/clear` — quality degrades beyond this |

## Provocation Prompts

When output quality drops:
- "Scrap this and implement the elegant solution"
- "Grill me on these changes"
- "Prove to me this works"
- "What would a senior engineer say?"
- "Is there a simpler way?"

## Quality Scoring (Phase Review)

| Dimension | Max | What it measures |
|-----------|-----|-----------------|
| Completeness | 20 | All deliverables built? |
| Code Quality | 20 | Clean architecture? |
| Test Coverage | 20 | Tests pass? Edge cases? |
| Design/UX | 20 | Looks right? Accessible? |
| Documentation | 20 | Docs updated? Lessons saved? |
| **Threshold** | **80** | **Below = fix before proceeding** |
