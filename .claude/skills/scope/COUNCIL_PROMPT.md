# Council Pressure-Test Prompt (Versioned)

> Used by `/scope` Phase 6 to invoke `/llm-council` once tech stack is confirmed.
> Edit with care — changes affect every new project's scope review.

**Version**: 1.0.0

---

## Trigger

`/scope` invokes the council exactly once per scope run, after Phase 5 (Business Framing) and tech-stack confirmation, and before writing `project-scope.json`.

If the user re-runs `/scope` later, the council re-fires (material change deserves review).

---

## Prompt template

The scope skill substitutes `{{...}}` placeholders with values gathered during the conversation, then invokes the `llm-council` skill with the resulting text.

```
Pressure-test this project scope from a senior-engineer / product-strategist perspective.

PROJECT
- Name: {{project_name}}
- TLDR: {{tldr}}
- Problem: {{problem_statement}}
- Target user: {{target_users}}
- Business rationale: {{business_rationale}}

CONSTRAINTS
- Timeline: {{constraints.timeline}}
- Team size: {{constraints.team_size}}
- Team experience: {{constraints.team_experience}}
- Budget/hosting: {{constraints.budget_constraints}} / {{constraints.hosting_constraints}}
- Dealbreakers: {{constraints.dealbreakers}}

PROPOSED MVP (V1)
{{features_v1_bulleted}}

PROPOSED STACK
- Frontend: {{tech_stack.frontend}}
- Backend: {{tech_stack.backend}}
- Database: {{tech_stack.database}}
- Auth: {{tech_stack.auth}}
- Hosting: {{tech_stack.hosting}}
- Additional services: {{tech_stack.additional_services}}

ANSWER THESE THREE QUESTIONS

1. Is the MVP cut realistic for {{constraints.team_size}} person(s) in {{constraints.timeline}} given the constraints above? If not, what cuts would make it realistic?

2. Is the target user specific enough that we'd know who to interview tomorrow, or is it too broad? Is there a sharper segmentation that would reduce risk?

3. Does the chosen stack match the constraints, or is there a meaningfully better option? Be concrete — name alternatives and the tradeoff. Don't recommend changes for taste; only flag if there's a real mismatch (e.g., picking a heavyweight stack for a 2-week MVP, picking an obscure framework for a junior team, picking a stack the team has zero experience with for a tight deadline).

RETURN
- top_concerns: array of 3 short bullets, ordered by severity
- stack_critique: 1–3 sentences, or "no critique — stack matches constraints"
- mvp_critique: 1–3 sentences, or "no critique — MVP is realistic"
- verdict: single word — `ship` (proceed as-is), `revise` (fix the flagged items first), or `rethink` (foundational issues — go back to scope)
```

---

## Output handling

The council's structured response is written to `project-scope.json` under the `council_review` block:

```json
"council_review": {
  "verdict": "ship | revise | rethink",
  "top_concerns": ["..."],
  "stack_critique": "...",
  "mvp_critique": "...",
  "action_taken": "accepted | modified | overrode",
  "reviewed_at": "<ISO 8601>"
}
```

The `action_taken` field is filled by `/scope` based on the user's response to the verdict (see Phase 6 in `SKILL.md`):
- `accepted` — verdict was `ship`, or user accepted suggested changes.
- `modified` — verdict was `revise` or `rethink`; user agreed and `/scope` looped back to revise the relevant phase (one revision pass).
- `overrode` — user rejected the council's flagged concerns and proceeded anyway. Concerns persist into `PROJECT_SPEC.md` `Caveats` block so they re-surface at phase-review.

---

## Cost guardrail

Council fires **once per scope run** (not on every edit during the conversation). Expected cost on Opus-tier: $0.50–$2 per invocation. Documented up-front in the scope conversation so the user knows what they're paying for.

---

## What if `/llm-council` is not installed?

If the skill is unavailable, `/scope` writes:

```json
"council_review": {
  "verdict": "skipped",
  "top_concerns": [],
  "stack_critique": "council skill not installed",
  "mvp_critique": "council skill not installed",
  "action_taken": "skipped",
  "reviewed_at": "<ISO 8601>"
}
```

…and warns the user that the pressure-test was skipped. Scope completes; spec proceeds normally.
