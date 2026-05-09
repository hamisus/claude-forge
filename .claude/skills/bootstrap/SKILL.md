---
name: bootstrap
description: Generate project-specific skills, hooks, rules, agents, and CLAUDE.md from the project spec
version: 1.0.0
author: Claude Forge
category: phase-a
phase: implementation-setup
---

# Project Bootstrap Skill

This skill takes your completed specification (output from `/spec`) and generates all of the Claude Code infrastructure needed to build the project successfully.

## Overview

A Claude Forge project runs best when Claude Code is configured specifically for that project — custom hooks, tailored skills, stack-specific rules, and agents.

Bootstrap generates:
- **Root `CLAUDE.md`** *(REPLACED)* — Project overview, how to run, key conventions, preserved workflow content (provocations, context thresholds, quality scoring)
- **Root `README.md`** *(REPLACED)* — Project name, mission, stack, run instructions, phase summary
- **Hooks** — Stack-specific linting, code generation, testing, notifications
- **Custom Skills** — Component generation, model creation, test writing
- **Custom Rules** — Code style and pattern rules by layer (frontend, backend, data)
- **Custom Agents** — Code reviewer and test writer with stack-specific checklists
- **Permissions** — Stack-specific CLI commands that are safe to auto-run
- **Cleanup** — `docs/workflow-guide.md` is inlined into the new `CLAUDE.md`, then deleted (its content lives in CLAUDE.md going forward)
- **Backups** — Pre-existing root `CLAUDE.md` / `README.md` are backed up to `.claude/backups/<file>.<timestamp>` before overwrite (see "Backup Safeguard" below)

**Prerequisites:** You must have run `/scope` and `/spec` first.

---

## How to Use

```
/bootstrap
```

The skill will:
1. Read PROJECT_SPEC.md and ARCHITECTURE.md
2. Analyze your tech stack
3. Generate all infrastructure files
4. Verify all files are valid (JSON, YAML, Markdown)
5. Confirm the system is ready to build

---

## Generated Files

Files are created at the **project root** (CLAUDE.md, README.md) and inside `.claude/`:

```
<project-root>/
├── CLAUDE.md                          (REPLACED: project overview + preserved workflow content)
├── README.md                          (REPLACED: project name, stack, run instructions, phases)
├── docs/
│   └── workflow-guide.md              (DELETED: content inlined into CLAUDE.md)
└── .claude/
    ├── settings.json                  (UPDATED: hooks + permissions)
    ├── backups/
    │   ├── CLAUDE.md.<timestamp>      (NEW: pre-existing root CLAUDE.md, if any)
    │   └── README.md.<timestamp>      (NEW: pre-existing root README.md, if any)
    ├── skills/
    │   ├── new-component.md           (NEW: component/screen generator)
    │   ├── new-model.md               (NEW: data model generator)
    │   ├── new-test.md                (NEW: test writer)
    │   └── [existing skills preserved]
    ├── rules/
    │   ├── frontend-code.md           (NEW: UI code rules)
    │   ├── backend-code.md            (NEW: API/logic code rules)
    │   ├── data-code.md               (NEW: database/model rules)
    │   ├── useful-tests.md            (PRESERVED: test quality rule from Forge)
    │   └── [existing rules preserved]
    ├── agents/
    │   ├── code-reviewer.md           (UPDATED: stack-specific review checklist)
    │   └── test-writer.md             (UPDATED: stack-specific test patterns)
    └── [other existing files preserved]
```

> **Note:** Bootstrap **replaces** the root `CLAUDE.md` and `README.md` that ship with the Forge starter. These starters describe Claude Forge itself; after bootstrap, the project owns those files. See "Backup Safeguard" below — pre-existing files are always backed up before overwrite.

---

## Generated File 1: CLAUDE.md (root)

### Purpose
The master project reference. Lives at the **project root**, not `.claude/`. Always read this first when jumping into a session.

### Source of content

The new CLAUDE.md is assembled from three sources:

1. **Project-specific content** — derived from `PROJECT_SPEC.md`, `ARCHITECTURE.md`, `PHASES.md` (sections below).
2. **Preserved workflow content** — extracted verbatim from the Forge starter using `<!-- BOOTSTRAP_PRESERVE_BEGIN: <name> -->` / `<!-- BOOTSTRAP_PRESERVE_END: <name> -->` markers. Required preserved blocks:
   - `provocation` — Provocation Prompts (from starter root `CLAUDE.md`)
   - `context-management` — Context Management rules (from starter root `CLAUDE.md`)
   - `compaction` — Compaction Instructions (from starter root `CLAUDE.md`)
   - `context-thresholds` — Context % thresholds table (from `docs/workflow-guide.md`)
   - `quality-scoring` — Quality Scoring rubric (from `docs/workflow-guide.md`)
3. **Removed content** — all "Claude Forge is a universal bootstrap…" prose, Getting Started instructions targeting the Forge `/scope`/`/spec`/`/bootstrap` flow, and any reference to `claude-forge` as the project identity.

### Sections

**Project Identity**
```markdown
# [Project Name]

**Mission:** [One sentence mission]

**Stack:** [Frontend] + [Backend] + [Database] + [Hosting]

**Where to Run:** [Run instructions, e.g. "npm run dev"]

**Live URL:** [If applicable]

**GitHub:** [Repository link if applicable]
```

**Quick Start**

Step-by-step to get the project running locally:
```markdown
## Getting Started

1. Clone the repo
2. Install dependencies: [command]
3. Set up environment: [what variables, how]
4. Seed database (if applicable): [command]
5. Start the dev server: [command]
6. Open [URL] and verify it loads
```

**Project Structure**

```markdown
## Project Structure

```
[project-root]/
├── src/
│   ├── frontend/               (if applicable)
│   │   ├── components/
│   │   ├── pages/
│   │   └── services/
│   ├── backend/                (if applicable)
│   │   ├── routes/
│   │   ├── models/
│   │   └── services/
│   └── [other structure]
├── database/
│   └── migrations/             (if applicable)
├── tests/
├── .claude/
│   ├── CLAUDE.md
│   ├── skills/
│   ├── rules/
│   └── agents/
└── README.md
```
```

**Key Conventions**

Document the rules that everything must follow:

```markdown
## Key Conventions

**Naming:**
- Components: PascalCase (e.g., UserProfile.tsx)
- Files: kebab-case (e.g., user-profile.tsx)
- Database tables: plural snake_case (e.g., users, user_sessions)

**Imports:**
- Absolute imports from @/ for src/
- Relative imports for local files in same directory

**Styling:**
- Tailwind utility classes (no custom CSS unless unavoidable)
- Color names from DESIGN_SYSTEM.md

**API Response Format:**
```json
{
  "success": true,
  "data": [...],
  "error": null
}
```

**Error Handling:**
- All errors logged to [logging service]
- User-facing errors are generic
- Dev errors include stack traces in logs only

**Testing:**
- TDD-default per `/dev` (silent skips forbidden; log `tdd-skip-reason` if genuinely opting out)
- Tests must be **useful** per `.claude/rules/useful-tests.md` — drive code through public entry points, assert observable behavior, would catch plausible mutations
- ≥1 end-to-end test per feature
- Documented edge cases from `PROJECT_SPEC.md` each have a test
- Tests live alongside source files
```

**Stack Justification**

Briefly explain why each technology was chosen:

```markdown
## Stack Decisions

**[Frontend Framework]** — [why chosen for this project]

**[Backend Runtime]** — [why chosen for this project]

**[Database]** — [why chosen for this project]

**[Hosting]** — [why chosen for this project]
```

**What NOT to Change**

Explicit list of architectural boundaries:

```markdown
## What NOT to Change

These are the immutable decisions. If you need to change them, raise an issue first:

- **Database schema structure** — Schema changes require migration planning
- **API response format** — Changing the response envelope breaks clients
- **Component prop contracts** — Components are shared across the app
- **Authentication system** — Affects all protected routes
- **Environment configuration** — Deployment depends on the current approach
- **Primary data models** — [List specific models that are foundational]
```

**Verification Strategy**

How to know the project is in good shape:

```markdown
## Verification Strategy

**Before committing:**
- [ ] `npm run lint` passes (or equivalent for your stack)
- [ ] `npm run test` passes (or equivalent)
- [ ] **Useful Tests Gate** passed (per `.claude/rules/useful-tests.md`) — `useful-tests-audit:` block present in commit body
- [ ] No console errors in dev tools
- [ ] Manual smoke test of the happy path

**Before deploying:**
- [ ] All above, plus:
- [ ] Staging environment passes full test suite
- [ ] Performance budget checks pass
- [ ] Security scan shows no critical issues
- [ ] Database migrations are reversible
```

**Context Management**

How to keep Claude in sync when context is tight:

```markdown
## Context Management

**Compaction Script:** When context gets tight, run compaction to refresh context:
- Resets focus to current task
- Re-injects stack and architecture fundamentals
- Lists all available skills and rules
- Clears old conversation history

**Session Handoff:** When starting a new session:
1. Read this CLAUDE.md
2. Review the open task in [issue tracker or task file]
3. Run `/compact` to refresh architectural context
4. Ask "What's the current state of [feature]?"

**Knowledge Consolidation:**
- Keep CLAUDE.md as the source of truth
- Update it when architectural decisions change
- Run verification on every commit to catch drift
```

**Prompting Techniques**

How to get the best results from Claude:

```markdown
## Prompting Tips

**For implementation tasks:**
- Provide the feature spec from PROJECT_SPEC.md
- Reference the relevant data models
- Mention the relevant rules (frontend, backend, etc.)

**For debugging:**
- Include error messages verbatim
- Show the code that's failing
- Include logs if available

**For architecture questions:**
- Reference ARCHITECTURE.md for context
- Explain what you've already tried
- Ask for the "why" not just the "how"

**For code review:**
- Ask if the code follows the rules in .claude/rules/
- Ask if it matches the architecture in ARCHITECTURE.md
- Ask if tests are sufficient for the change
```

---

## Generated File 2: README.md (root)

### Purpose
The public-facing project reference. Lives at the **project root**, replaces the Forge starter README that describes Claude Forge itself.

### Source of content
- **Name + tagline** — from `project_name` and `tldr` in `project-scope.json`.
- **What it does** — from `description` and `vision_statement` in `project-scope.json`.
- **Stack** — from `tech_stack` in `project-scope.json` and the Stack Rationale section of `ARCHITECTURE.md`.
- **Run instructions** — from CLAUDE.md's Getting Started section (same commands; README mirrors them for new contributors who haven't read CLAUDE.md yet).
- **Phase summary** — extracted from `PHASES.md` (one line per phase: name + deliverable).

### Template

```markdown
# {{project_name}}

> {{tldr}}

{{description}}

## Stack

- **Frontend:** {{tech_stack.frontend}}
- **Backend:** {{tech_stack.backend}}
- **Database:** {{tech_stack.database}}
- **Auth:** {{tech_stack.auth}}
- **Hosting:** {{tech_stack.hosting}}

See `.claude/specs/ARCHITECTURE.md` for the full Stack Rationale (researched library choices with tradeoffs).

## Getting Started

```bash
# 1. Clone
git clone {{repo_url}} && cd {{project_slug}}

# 2. Install
{{install_command}}

# 3. Configure
cp .env.example .env  # then fill in values

# 4. Run
{{dev_command}}
```

Then open {{local_url}}.

## Development

This project uses [Claude Forge](https://github.com/hamisus/claude-forge) workflow conventions:

- **`/dev <feature>`** — plan → TDD → verify loop (TDD-default per `.claude/rules/useful-tests.md`)
- **`/wrap-up`** — end a session: commit, capture lessons, update handoff
- **`/phase-review`** — phase audit + 5-dimension quality score (≥80 to ship)

See `CLAUDE.md` for the full workflow guide and `.claude/specs/` for the project's specification suite.

## Project Phases

{{phase_summary_from_PHASES_md}}

## License

{{license}}
```

### Removed content (vs. Forge starter)

- All "Claude Forge is a universal bootstrap" content
- The "Quick Start" section that points at `/scope` → `/spec` → `/bootstrap`
- The "What's Included" directory listing of the Forge template

---

## Backup Safeguard

Replacing root `CLAUDE.md` and `README.md` is **destructive**. Before any overwrite:

1. **Detect divergence** — compute sha256 of the current root `CLAUDE.md` / `README.md` and compare against the known-starter manifest (sha256 of the file as shipped in Claude Forge).
2. **Always back up** — copy to `.claude/backups/<file>.<ISO-timestamp>` regardless of divergence. Cheap insurance.
3. **Refuse on hand-edits without `--force`** — if the sha256 differs from the starter (i.e., the file has been hand-edited), bootstrap halts and prints:
   ```
   Root CLAUDE.md has been hand-edited (sha256 differs from Forge starter).
   The original has been backed up to .claude/backups/CLAUDE.md.<timestamp>.
   To proceed and overwrite anyway, re-run with: /bootstrap --force
   To preserve hand-edited content, copy it manually from the backup before re-running.
   ```
4. **`--force` proceeds anyway** — backup is still written. User accepts that hand-edits are lost in the new file but recoverable from `.claude/backups/`.
5. **Backup retention** — bootstrap does not prune. User manages `.claude/backups/` size manually (it's small — two markdown files per re-bootstrap).

The `docs/workflow-guide.md` deletion follows the same pattern: backed up to `.claude/backups/workflow-guide.md.<timestamp>` before deletion.

### Known starter sha256

To detect divergence, bootstrap embeds the sha256 of the Forge starter `CLAUDE.md` and `README.md` (computed at Forge release time). Re-bootstrapping after pulling a new Forge version updates these manifests; bootstrap warns if the local Forge version is out of date.

---

## Generated File 3: settings.json

### Purpose
Configure Claude Code behavior with stack-specific hooks and permissions.

### Hooks

**PostToolUse Hook** — Runs after every command execution to catch errors immediately

Stack-specific examples:

```json
"postToolUse": {
  "enabled": true,
  "commands": [
    {
      "trigger": "file_write",
      "condition": "file ends with .ts or .tsx",
      "run": "eslint --fix <file_path> 2>&1",
      "description": "TypeScript linter"
    },
    {
      "trigger": "file_write",
      "condition": "file ends with .py",
      "run": "ruff check --fix <file_path> 2>&1",
      "description": "Python linter"
    },
    {
      "trigger": "file_write",
      "condition": "file is pubspec.yaml",
      "run": "flutter pub get",
      "description": "Dart dependency refresh"
    },
    {
      "trigger": "file_write",
      "condition": "file is schema.prisma",
      "run": "prisma generate",
      "description": "Prisma code generation"
    }
  ]
}
```

**PreCommit Hook** — Runs before every commit

```json
"preCommit": {
  "enabled": true,
  "commands": [
    {
      "run": "npm run lint && npm run test 2>&1",
      "description": "Lint and test before commit"
    }
  ]
}
```

**Compact Hook** — Refreshes context when memory is tight

```json
"compact": {
  "enabled": true,
  "instructions": "Inject the following when context resets:\n\n# Quick Context Refresh\n\n**Stack:** [Frontend] + [Backend] + [Database]\n\n**Key Rules:**\n- [Rule 1]\n- [Rule 2]\n- [Rule 3]\n\n**Available Skills:**\n- /new-component — Create new UI component\n- /new-model — Create new data model\n- /new-test — Write new test\n\n**Current Status:** [last verified state]\n\nRun `cat .claude/CLAUDE.md` to re-read full project overview."
}
```

**Notification Hook** — Alerts when important events happen

```json
"notification": {
  "enabled": true,
  "events": [
    {
      "trigger": "test_failure",
      "message": "Test suite failed. Review error above."
    },
    {
      "trigger": "lint_failure",
      "message": "Linting errors detected. Run linter to fix."
    }
  ]
}
```

**Stop Hook** — Runs before session ends

```json
"stop": {
  "enabled": true,
  "commands": [
    {
      "run": "[Verification checklist command]",
      "description": "Verify project state before ending session"
    }
  ]
}
```

### Permissions

Safe stack-specific commands that can auto-run:

```json
"permissions": {
  "allowedCommands": [
    "npm run lint",
    "npm run lint -- --fix",
    "npm run test",
    "npm run build",
    "npx eslint .",
    "npx prettier --write .",
    "python -m pytest",
    "python -m black .",
    "flutter analyze",
    "flutter test",
    "go fmt ./...",
    "go test ./...",
    "cargo fmt",
    "cargo test",
    "docker compose up -d",
    "git status",
    "git diff",
    "git add",
    "git commit",
    "git log --oneline -20"
  ]
}
```

---

## Generated File 4: new-component.md

### Purpose
Skill that generates new components/screens following stack conventions.

### Template Structure

For a React+TypeScript project:

```yaml
---
name: new-component
description: Generate a new React component following project conventions
version: 1.0.0
---

# New Component Generator

Generates new UI components following project patterns.

## How to Use

```
/new-component ComponentName
```

## What Gets Generated

```
src/components/component-name/
├── ComponentName.tsx          (Component implementation)
├── ComponentName.module.css   (Scoped styles)
├── ComponentName.test.tsx     (Test file)
└── index.ts                   (Export)
```

## Pattern

**ComponentName.tsx:**
```typescript
import React from 'react';
import styles from './ComponentName.module.css';

interface ComponentNameProps {
  // Define props here
}

export const ComponentName: React.FC<ComponentNameProps> = (props) => {
  // Component implementation
  return <div className={styles.container}>{/* content */}</div>;
};
```

**Testing:**
- Each component has a .test.tsx file
- Tests cover happy path and error states
- Use React Testing Library patterns

**Styling:**
- Use CSS modules for scoped styles
- Use design system colors and spacing
- Responsive breakpoints: mobile-first

**Documentation:**
- JSDoc comments on complex components
- Storybook stories for reusable components
```

For a Flutter project:

```yaml
---
name: new-component
description: Generate a new Flutter widget following project conventions
version: 1.0.0
---

# New Widget Generator

Generates new Flutter widgets following project patterns.

## How to Use

```
/new-component WidgetName
```

## What Gets Generated

```
lib/widgets/widget_name/
├── widget_name.dart          (Widget implementation)
├── widget_name_test.dart     (Test file)
└── index.dart                (Export)
```

## Pattern

**widget_name.dart:**
```dart
import 'package:flutter/material.dart';

class WidgetName extends StatefulWidget {
  final String title;

  const WidgetName({
    Key? key,
    required this.title,
  }) : super(key: key);

  @override
  State<WidgetName> createState() => _WidgetNameState();
}

class _WidgetNameState extends State<WidgetName> {
  @override
  Widget build(BuildContext context) {
    return Container();
  }
}
```

**Testing:**
- Use testWidgets for widget tests
- Test user interactions and state changes
- Use WidgetTester.

**Styling:**
- Use theme colors from design_system
- Responsive with MediaQuery or LayoutBuilder
```

---

## Generated File 5: new-model.md

### Purpose
Skill that generates new data models following database conventions.

### Template for SQL Database

```yaml
---
name: new-model
description: Generate a new data model and database migration
version: 1.0.0
---

# New Data Model Generator

Generates new data models with database migrations.

## How to Use

```
/new-model ModelName [fields]
```

Example:
```
/new-model Article title:string content:text author_id:uuid published:boolean
```

## What Gets Generated

**Prisma Schema** (if using Prisma):
```prisma
model Article {
  id        String   @id @default(cuid())
  title     String
  content   String   @db.Text
  authorId  String
  author    User     @relation(fields: [authorId], references: [id])
  published Boolean  @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([authorId])
}
```

**Migration:**
```sql
-- Generated migration
CREATE TABLE articles (
  id TEXT PRIMARY KEY,
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  author_id TEXT NOT NULL REFERENCES users(id),
  published BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_articles_author_id ON articles(author_id);
```

**API Routes** (generated):
```
GET    /api/articles              — List all articles
POST   /api/articles              — Create new article
GET    /api/articles/{id}         — Get single article
PUT    /api/articles/{id}         — Update article
DELETE /api/articles/{id}         — Delete article
```

**Type Definition:**
```typescript
export interface Article {
  id: string;
  title: string;
  content: string;
  authorId: string;
  published: boolean;
  createdAt: Date;
  updatedAt: Date;
}
```

**Test Seeds:**
```javascript
export const articleFixtures = {
  published: {
    title: 'Example Article',
    content: 'Article content',
    authorId: 'user-1',
    published: true
  },
  draft: {
    title: 'Draft Article',
    content: 'Work in progress',
    authorId: 'user-1',
    published: false
  }
};
```
```

---

## Generated File 6: new-test.md

### Purpose
Skill that generates new tests following stack conventions.

### Template for JavaScript/TypeScript

```yaml
---
name: new-test
description: Generate a new test file following project patterns
version: 1.0.0
---

# New Test Generator

Generates test files for the current stack.

## How to Use

```
/new-test path/to/file.ts
```

## Generated Test Pattern

For **unit tests** (business logic):

```typescript
import { describe, it, expect, beforeEach } from 'vitest';
import { calculateTotal } from './calculate-total';

describe('calculateTotal', () => {
  it('should sum array of prices', () => {
    const result = calculateTotal([10, 20, 30]);
    expect(result).toBe(60);
  });

  it('should return 0 for empty array', () => {
    const result = calculateTotal([]);
    expect(result).toBe(0);
  });

  it('should handle decimal prices', () => {
    const result = calculateTotal([10.50, 20.25]);
    expect(result).toBeCloseTo(30.75);
  });
});
```

For **integration tests** (API endpoints):

```typescript
import { describe, it, expect } from 'vitest';
import { request } from 'supertest';
import app from '@/app';

describe('POST /api/users', () => {
  it('should create a new user', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ email: 'test@example.com', name: 'Test User' });

    expect(response.status).toBe(201);
    expect(response.body.data.email).toBe('test@example.com');
  });

  it('should reject duplicate email', async () => {
    await request(app)
      .post('/api/users')
      .send({ email: 'test@example.com', name: 'User 1' });

    const response = await request(app)
      .post('/api/users')
      .send({ email: 'test@example.com', name: 'User 2' });

    expect(response.status).toBe(409);
  });
});
```

For **component tests** (React):

```typescript
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import { Button } from './Button';

describe('Button', () => {
  it('should render with label', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('should call onClick when clicked', () => {
    const onClick = vi.fn();
    render(<Button onClick={onClick}>Click me</Button>);

    fireEvent.click(screen.getByText('Click me'));
    expect(onClick).toHaveBeenCalled();
  });
});
```

**Coverage Target:** 80%+ line coverage for business logic
```

---

## Generated File 7: frontend-code.md

### Purpose
Rules for frontend/UI code.

### Template

```yaml
---
name: frontend-code
description: Code rules for frontend/UI code
---

# Frontend Code Rules

## Components

- [ ] Props are typed with interfaces or props objects
- [ ] Components have clear, descriptive names (PascalCase)
- [ ] Props interface includes JSDoc for complex props
- [ ] Single Responsibility — each component does one thing well
- [ ] No business logic in components; use hooks or services
- [ ] Reusable components go in shared /components folder
- [ ] Page-specific components live in their page folder

## State Management

- [ ] Local state with useState for UI state only
- [ ] Global state via [Redux/Context/Zustand] for shared business state
- [ ] State shape mirrors data model structure
- [ ] Avoid prop drilling; use context or state management
- [ ] Selectors are memoized to prevent unnecessary re-renders

## Styling

- [ ] Use Tailwind utilities; avoid custom CSS except for complex layouts
- [ ] Color values come from design system (not hardcoded)
- [ ] Spacing uses design system scale (sm, md, lg, xl)
- [ ] Responsive classes use mobile-first breakpoints: sm:, md:, lg:, xl:
- [ ] Hover/focus/active states are defined for interactive elements
- [ ] Dark mode support if applicable

## Events & Forms

- [ ] Form validation happens on blur and submit
- [ ] Errors display near the field that failed
- [ ] Success feedback is clear (toast, banner, or visual change)
- [ ] Submit buttons are disabled during submission
- [ ] Loading states are shown for async operations

## Testing

- [ ] Components have test files alongside source
- [ ] Happy path is tested
- [ ] Error states are tested
- [ ] User interactions (click, type, submit) are tested
- [ ] Use React Testing Library (test behavior, not implementation)

## Accessibility

- [ ] Semantic HTML (buttons are <button>, links are <a>)
- [ ] Form labels are associated with inputs
- [ ] Color is not the only indicator of state (use icons, text, patterns)
- [ ] Focus indicators are visible (not removed)
- [ ] Interactive elements have min 44x44px touch target
- [ ] Alt text on images
- [ ] Heading hierarchy is correct (no skipping h1 -> h3)

## Performance

- [ ] Images are optimized and lazy-loaded
- [ ] Long lists use virtualization (e.g., react-window)
- [ ] Expensive computations are memoized (useMemo, useCallback)
- [ ] No console warnings or errors in dev tools
- [ ] Bundle size is monitored
```

---

## Generated File 8: backend-code.md

### Purpose
Rules for backend/API code.

### Template

```yaml
---
name: backend-code
description: Code rules for backend/API code
---

# Backend Code Rules

## API Routes

- [ ] Routes are organized by resource (e.g., /routes/users.ts, /routes/posts.ts)
- [ ] Naming: GET, POST, PUT, DELETE with clear paths
- [ ] Request validation happens before business logic
- [ ] All responses follow standard format: `{ success, data, error }`
- [ ] Errors include appropriate HTTP status codes
- [ ] Rate limiting is applied to sensitive endpoints

## Business Logic

- [ ] Service layer separates business logic from routes
- [ ] Services handle data transformation and validation
- [ ] Services don't touch HTTP (routes do)
- [ ] Complex operations are documented
- [ ] Error handling provides context (what failed, why)
- [ ] All external service calls have retry logic

## Database

- [ ] All database operations go through a repository or ORM
- [ ] No raw SQL in business logic
- [ ] Queries are indexed for performance
- [ ] N+1 queries are avoided (use eager loading)
- [ ] Migrations are reversible
- [ ] Schema changes are backward-compatible when possible

## Authentication

- [ ] All protected routes verify token/session
- [ ] Permissions are checked (not just authentication)
- [ ] Sensitive operations log who did what
- [ ] Tokens expire and are refreshed securely
- [ ] Passwords are hashed (never stored plaintext)

## Testing

- [ ] All routes have integration tests
- [ ] All services have unit tests
- [ ] Tests use fixtures/factories for data
- [ ] Error cases are tested
- [ ] Database state is reset between tests

## Security

- [ ] User input is validated and sanitized
- [ ] SQL injection is prevented (use ORM or prepared statements)
- [ ] CORS is configured correctly
- [ ] Sensitive data (passwords, tokens) is never logged
- [ ] Environment variables are used for secrets
- [ ] HTTPS is enforced in production

## Logging

- [ ] Errors are logged with context (request ID, timestamp, stack)
- [ ] Sensitive data is not logged
- [ ] Log levels are appropriate (error, warn, info, debug)
- [ ] Logs are aggregated and searchable

## Performance

- [ ] Database queries are optimized
- [ ] Caching is used for frequent queries
- [ ] Pagination is implemented for large result sets
- [ ] Compression is enabled for responses
- [ ] Timeouts are set for all external calls
```

---

## Generated File 9: data-code.md

### Purpose
Rules for data models and database code.

### Template

```yaml
---
name: data-code
description: Code rules for data models and database schemas
---

# Data & Database Rules

## Schema Design

- [ ] Tables use snake_case names, plural (users, user_sessions)
- [ ] Primary keys are UUID or BIGINT (not sequential if distributed)
- [ ] Foreign keys reference correct tables and have indexes
- [ ] NOT NULL constraints on required fields
- [ ] UNIQUE constraints on email, username, identifiers
- [ ] Timestamps: created_at, updated_at on all tables
- [ ] Soft deletes (deleted_at) if needed, or hard deletes only

## Migrations

- [ ] Each migration is single concern (one table or related changes)
- [ ] Migrations are reversible (UP and DOWN)
- [ ] Schema changes are backward-compatible when possible
- [ ] Data migrations are separate from schema migrations
- [ ] Migrations are tested before deploy
- [ ] Naming is descriptive: 001_create_users.sql, 002_add_email_index.sql

## Indexes

- [ ] Foreign key columns are indexed
- [ ] Frequently filtered/searched columns are indexed
- [ ] Composite indexes match query patterns
- [ ] Indexes are monitored for unused or redundant indexes
- [ ] Index creation doesn't block production traffic

## Data Validation

- [ ] Type constraints in schema (INT, VARCHAR, etc.)
- [ ] Business logic constraints in application layer
- [ ] Required fields have NOT NULL
- [ ] String length constraints if applicable
- [ ] Check constraints for enums or ranges

## Relationships

- [ ] One-to-Many relationships use foreign keys
- [ ] Many-to-Many relationships use junction tables
- [ ] Cascade deletes are intentional and documented
- [ ] Orphan records are prevented or cleaned up
- [ ] Referential integrity is enforced at database level

## Backup & Recovery

- [ ] Backups run daily (or per SLA)
- [ ] Backup retention matches compliance requirements
- [ ] Recovery procedures are tested monthly
- [ ] Point-in-time recovery is possible
- [ ] Backups are encrypted and stored off-site
```

---

## Generated File 10: code-reviewer.md

### Purpose
Custom code review checklist tailored to the stack.

### Template Update

```yaml
---
name: code-reviewer
description: Stack-specific code review guidance
---

# Code Reviewer Agent

When reviewing code, check:

## Architecture & Design

- [ ] Code follows patterns in ARCHITECTURE.md
- [ ] No architectural boundaries are crossed
- [ ] Complex logic is in services/helpers, not components
- [ ] Error handling is consistent with project standards

## Stack-Specific Checks (React)

- [ ] Components follow naming and structure conventions
- [ ] No unnecessary re-renders (proper memoization)
- [ ] Props are typed
- [ ] Hooks are used correctly (dependencies array, rules of hooks)
- [ ] No external API calls in components (use effects or services)

## Stack-Specific Checks (Python/FastAPI)

- [ ] Routes are organized by resource
- [ ] Type hints are present
- [ ] Exceptions are caught and logged
- [ ] Async/await is used correctly
- [ ] Database queries are optimized

## Testing

- [ ] Tests cover happy path and error cases
- [ ] Mocks are used appropriately
- [ ] Tests are deterministic (no flakiness)
- [ ] Coverage is above 80% for business logic

## Security

- [ ] User input is validated
- [ ] Sensitive data is not logged
- [ ] Auth checks are present on protected operations
- [ ] No hardcoded secrets or credentials

## Performance

- [ ] Database queries are indexed
- [ ] N+1 queries are avoided
- [ ] Expensive operations are cached or memoized
- [ ] No unnecessary re-renders or re-fetches

## Documentation

- [ ] Complex functions have JSDoc/docstrings
- [ ] Async behavior is documented
- [ ] Dependencies are explained
```

---

## Generated File 11: test-writer.md

### Purpose
Custom test writing guidance tailored to the stack.

### Template Update

```yaml
---
name: test-writer
description: Stack-specific test writing patterns
---

# Test Writer Agent

When writing tests, follow these patterns:

## Unit Tests

Test individual functions/services in isolation:

```typescript
describe('calculateDiscount', () => {
  it('should apply percentage discount', () => {
    expect(calculateDiscount(100, 10)).toBe(90);
  });

  it('should return original price for 0% discount', () => {
    expect(calculateDiscount(100, 0)).toBe(100);
  });

  it('should handle negative prices', () => {
    expect(calculateDiscount(-100, 10)).toBe(-90);
  });
});
```

## Integration Tests

Test API routes with realistic data flows:

```typescript
describe('POST /api/orders', () => {
  it('should create order with valid data', async () => {
    const response = await request(app)
      .post('/api/orders')
      .send({ items: [...], userId: '123' });

    expect(response.status).toBe(201);
    expect(response.body.data.id).toBeDefined();
  });

  it('should fail with insufficient inventory', async () => {
    // Setup: insufficient stock
    const response = await request(app)
      .post('/api/orders')
      .send({ items: [{ productId: 'x', qty: 1000 }] });

    expect(response.status).toBe(400);
    expect(response.body.error).toContain('inventory');
  });
});
```

## Component Tests

Test UI components with user interactions:

```typescript
describe('UserForm', () => {
  it('should submit with valid data', async () => {
    const onSubmit = vi.fn();
    render(<UserForm onSubmit={onSubmit} />);

    fireEvent.change(screen.getByLabelText('Name'), { target: { value: 'John' } });
    fireEvent.click(screen.getByText('Submit'));

    expect(onSubmit).toHaveBeenCalledWith({ name: 'John' });
  });

  it('should show error for empty name', async () => {
    render(<UserForm />);
    fireEvent.click(screen.getByText('Submit'));

    expect(screen.getByText('Name is required')).toBeInTheDocument();
  });
});
```

## Test Structure

- **Arrange:** Set up test data and mocks
- **Act:** Call the function or interact with component
- **Assert:** Verify the result

## Fixtures & Factories

Use consistent test data:

```typescript
const userFixture = {
  id: 'user-1',
  email: 'test@example.com',
  name: 'Test User'
};

const createUser = (overrides = {}) => ({
  ...userFixture,
  ...overrides
});
```

## Coverage Targets

- **Business logic:** 80%+ line coverage
- **Components:** Happy path + error states
- **API routes:** All endpoints and error cases
- **Critical paths:** 100% for payment, auth, data deletion
```

---

## Verification & Final Steps

After generating all files, the skill performs these checks:

**File Validity:**
- [ ] All .json files are valid JSON
- [ ] All .md files are valid Markdown
- [ ] All .yaml files are valid YAML
- [ ] No syntax errors in code samples

**Completeness:**
- [ ] Root `CLAUDE.md` replaced (no "Claude Forge" references)
- [ ] Root `CLAUDE.md` contains preserved blocks: provocation, context-management, compaction, context-thresholds, quality-scoring
- [ ] Root `README.md` replaced (project-named, not "Claude Forge")
- [ ] `docs/workflow-guide.md` deleted (content present in `CLAUDE.md`)
- [ ] `.claude/backups/` contains pre-existing CLAUDE.md, README.md, workflow-guide.md
- [ ] `.claude/rules/useful-tests.md` is present (preserved from Forge starter)
- [ ] All four custom skills are generated
- [ ] All three custom rules are generated
- [ ] Both custom agents are generated
- [ ] settings.json has all hooks
- [ ] settings.json has permissions for stack

**Consistency:**
- [ ] Tech stack in `CLAUDE.md` matches `ARCHITECTURE.md`
- [ ] Tech stack in `README.md` matches `CLAUDE.md`
- [ ] All skill references match actual file locations
- [ ] All hook commands are valid for the stack
- [ ] All generated code samples match project conventions
- [ ] `CLAUDE.md` references `.claude/rules/useful-tests.md` in its Testing section

**Ready to Build:**
- [ ] All files are in place
- [ ] Project structure is clear
- [ ] Development workflow is defined (TDD-default, useful-tests gate)
- [ ] Code patterns are documented

---

## Output Summary

When complete, bootstrap outputs:

```
✓ Backed up CLAUDE.md → .claude/backups/CLAUDE.md.<timestamp>
✓ Backed up README.md → .claude/backups/README.md.<timestamp>
✓ Backed up docs/workflow-guide.md → .claude/backups/workflow-guide.md.<timestamp>
✓ Replaced root CLAUDE.md (project-specific + preserved workflow content)
✓ Replaced root README.md (project-specific)
✓ Deleted docs/workflow-guide.md (content inlined into CLAUDE.md)
✓ Updated .claude/settings.json (hooks + permissions)
✓ Generated .claude/skills/new-component.md
✓ Generated .claude/skills/new-model.md
✓ Generated .claude/skills/new-test.md
✓ Generated .claude/rules/frontend-code.md
✓ Generated .claude/rules/backend-code.md
✓ Generated .claude/rules/data-code.md
✓ Preserved .claude/rules/useful-tests.md (test quality rule)
✓ Updated .claude/agents/code-reviewer.md
✓ Updated .claude/agents/test-writer.md

Your project is bootstrapped and ready to build!

Next steps:
1. Review root CLAUDE.md to understand project setup and workflow
2. Run the project locally with the command in CLAUDE.md
3. Start building with /dev <feature> (TDD-default per .claude/rules/useful-tests.md)
4. Optional: /kanban-prep to export parallel-safe tasks for Cline Kanban
```

---

## Ready?

Run `/bootstrap` to generate your complete Claude Code infrastructure.

After bootstrap, your project is ready for development. All custom skills, rules, and agents are in place and tailored to your stack and architecture.

Start building with `/dev` or reference a specific feature from PHASES.md.
