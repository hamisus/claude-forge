---
name: "new-feature"
description: "Generic feature creation template. Stack-agnostic placeholder. After /bootstrap runs, you'll have stack-specific skills (/new-component, /new-endpoint, etc.) that replace this."
---

# /new-feature — Generic Feature Creation

This is a **generic, stack-agnostic placeholder** for creating new features.

## ⚠️ Important Note
This skill is intentionally generic because the Claude Forge system is **stack-agnostic** — it works with any language, framework, or architecture.

If you haven't run `/bootstrap` yet:
- You're seeing this generic skill
- It provides a basic template for new features
- It will be supplemented by stack-specific skills after `/bootstrap` runs

If you have run `/bootstrap`:
- You should have stack-specific skills like:
  - `/new-component` (for React, Vue, etc.)
  - `/new-endpoint` (for REST APIs, GraphQL, etc.)
  - `/new-screen` (for Flutter, SwiftUI, etc.)
  - `/new-job` (for background jobs, cron tasks, etc.)
  - etc.
- Use those skills instead of this generic one
- They're tailored to your specific stack

## Generic Feature Creation Steps

### 1. UNDERSTAND THE FEATURE REQUEST
- What problem does this feature solve?
- What does "done" look like?
- Any constraints (performance, design, compatibility)?

### 2. PLAN THE FEATURE
Use the `/dev` skill's planning approach:
- Break into small logical units
- Present plan to user with annotation cycle
- Get approval before building

### 3. CREATE FEATURE DIRECTORY
Most features need a dedicated directory:
```
src/
  features/
    new-feature-name/
      index.ts              (main export)
      implementation.ts     (main logic)
      tests.ts              (tests)
      CLAUDE.md             (feature-specific lessons)
```

### 4. CREATE MAIN FILE
Create the primary implementation file:
- Name it something meaningful (not generic.ts)
- Add comments explaining intent
- Follow existing code style

### 5. CREATE TEST FILE
Write tests first (TDD approach, use `/tdd` skill):
- Define expected behavior
- Edge cases
- Error cases

### 6. ADD ROUTE/REGISTRATION
Most features need to be registered in the system:
- Add route to router/routes file
- Add export to main index.ts
- Add configuration if needed

### 7. VERIFY
- Build passes (no syntax errors)
- Tests pass (all green)
- Feature is accessible/callable
- No import errors

### 8. USE /dev FOR OVERALL WORKFLOW
This skill is meant to be a quick reference. For actually building the feature:
- Use `/dev` for master development workflow
- Use stack-specific skills when available
- Use `/tdd` for test-driven development
- Use `/debug` if something breaks

## Example: Building a Feature with Stack-Specific Skills

**If you have React skills** (after /bootstrap):
```
User: "Build a search UI component"
SKILL: "I see we're using React. Using /new-component instead."
SKILL: [calls /new-component skill]
SKILL: "Creating SearchBox component with tests, styling, integration..."
```

**If you have Node.js/Express skills** (after /bootstrap):
```
User: "Build a user API endpoint"
SKILL: "I see we're using Express. Using /new-endpoint instead."
SKILL: [calls /new-endpoint skill]
SKILL: "Creating GET /api/users endpoint with validation, tests..."
```

**If neither stack-specific skill exists** (fall back to generic):
```
User: "Build a feature"
SKILL: "I don't have a stack-specific skill for this. Using generic /new-feature."
SKILL: [uses steps above]
```

## When to Use This Skill

✅ **Use /new-feature when:**
- You're building a new feature and haven't run `/bootstrap` yet
- You're building a feature type not covered by stack-specific skills
- You need a quick generic reference

❌ **Don't use /new-feature when:**
- A stack-specific skill exists (`/new-component`, `/new-endpoint`, etc.)
- You're using `/dev` for overall workflow (reference that instead)
- You're building tests (use `/tdd` instead)

## What Happens After /bootstrap?

After `/bootstrap` runs:
- Your stack is detected (React, Node.js, Django, Flutter, etc.)
- Stack-specific skills are created:
  - `/new-component` — for UI components
  - `/new-endpoint` — for API endpoints
  - `/new-job` — for background tasks
  - `/new-screen` — for mobile screens
  - etc.
- These skills replace this generic one
- Use stack-specific skills for all feature creation going forward

Example enhanced skill after /bootstrap:
```
# /new-component (React-specific)

This skill creates a new React component with:
- Component template (functional, hooks)
- CSS module or styled-components
- Unit tests using Jest/React Testing Library
- Storybook entry (optional)
- Auto-registration in component index

Usage: Creates component scaffolding for a new component
```

## Directory Structure Reference

After creating a feature, your structure might look like:

**Option A: Feature-based**
```
src/
  features/
    search/
      components/
        SearchBox.tsx
        SearchResults.tsx
      api.ts
      types.ts
      tests/
        SearchBox.test.tsx
        api.test.ts
      CLAUDE.md
```

**Option B: Layer-based**
```
src/
  components/
    SearchBox.tsx
  api/
    search.ts
  services/
    searchService.ts
  tests/
    search.test.ts
```

**Option C: Custom**
Structure depends on your project. Check `CLAUDE.md` or existing patterns.

## Integration with Other Skills
- **`/dev`**: Use this for overall feature workflow
- **`/tdd`**: Use for test-first development
- **`/debug`**: Use if something breaks
- **`/learn`**: Use to save lessons
- **Stack-specific skills**: Use instead of this generic skill when available

## Checklist (Generic Approach)
- [ ] Understood feature requirements
- [ ] Planned feature with user approval
- [ ] Created feature directory
- [ ] Created main implementation file
- [ ] Created test file with tests first
- [ ] Added route/registration
- [ ] Build passes (no errors)
- [ ] Tests pass (all green)
- [ ] Feature works end-to-end

## Key Principles
1. **This is a placeholder**: Stack-specific skills are better
2. **Run /bootstrap**: Gets you real, tailored skills
3. **Use /dev**: For actual feature workflow
4. **Tests first**: Use `/tdd` skill
5. **Stack-agnostic**: Works with any language/framework, but less tailored than stack-specific skills

## Next Steps
1. If you haven't run `/bootstrap` yet, do so now
2. Once `/bootstrap` creates stack-specific skills, use those instead
3. Use `/dev` for overall feature workflow
4. Use stack-specific skills for details and boilerplate
