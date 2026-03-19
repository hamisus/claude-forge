---
name: spec
description: Generate project specification, architecture, phases, and design system from scope
version: 1.0.0
author: Claude Forge
category: phase-a
phase: planning
---

# Project Specification Skill

This skill takes your scoped project (output from `/scope`) and generates comprehensive specification and architecture documents that guide implementation.

## Overview

A good spec doesn't over-document. It provides just enough detail for a Claude Code session to implement features without guessing. This skill generates four focused documents:

1. **PROJECT_SPEC.md** — Feature definitions, data models, API routes, user flows
2. **ARCHITECTURE.md** — System design, layer boundaries, data flow, immutable decisions
3. **PHASES.md** — Buildable work phases, each sized for 1-3 sessions
4. **DESIGN_SYSTEM.md** *(UI projects only)* — Visual language, components, patterns

**Prerequisite:** You must have run `/scope` first to generate `.claude/project-scope.json`.

---

## How to Use

```
/spec
```

The skill will:
1. Read `.claude/project-scope.json`
2. Ask clarifying questions about features, data, and architecture
3. Generate the four spec documents in `.claude/specs/`
4. Verify all documents are complete and consistent

---

## Output Structure

All spec documents are saved to `.claude/specs/`:

```
.claude/specs/
├── PROJECT_SPEC.md
├── ARCHITECTURE.md
├── PHASES.md
└── DESIGN_SYSTEM.md (if UI project)
```

---

## Document 1: PROJECT_SPEC.md

### Purpose
Comprehensive feature specification. This is the reference document for what you're building.

### Sections

**Project Overview**
- Project name, mission statement
- One-paragraph description
- Target users and their key needs

**Feature Specifications (v1 & v2)**

For each feature:
```
## Feature Name

**User Story:** "As a [user], I want to [action], so that [outcome]"

**Description:** Detailed explanation of what this feature does and why it matters.

**User Flow:**
1. User arrives at [location]
2. User sees [UI elements]
3. User performs [action]
4. System responds with [result]

**Data Dependencies:** What data must exist? What gets created/modified?

**Edge Cases:**
- What if user is offline?
- What if data is stale?
- What if permission is denied?

**Success Criteria:**
- User can [action]
- System correctly [behavior]
- Performance: [requirement]

**Metrics:** How will success be measured?
```

**Data Models**

```
## Data Models

### User
- id (uuid, primary key)
- email (string, unique, required)
- name (string)
- created_at (timestamp)
- updated_at (timestamp)
- [relationships to other models]

### [Entity]
[similar structure]
```

**API Routes/Endpoints** (if applicable)

```
## API Endpoints

### POST /api/users
- **Purpose:** Create a new user
- **Auth:** Public
- **Request Body:**
  ```json
  {
    "email": "string",
    "name": "string"
  }
  ```
- **Response:** User object, 201 Created
- **Errors:** 400 (validation), 409 (duplicate email)

### GET /api/users/{id}
[similar detail]
```

**Authentication & Authorization**

- Who can do what?
- User roles/permissions matrix
- Session/token handling
- Public vs. protected endpoints

**Third-Party Integrations**

For each integration:
- Purpose and criticality
- Authentication method
- Rate limits
- Error handling strategy
- Fallback behavior if service is down

---

## Document 2: ARCHITECTURE.md

### Purpose
System design guide. Answers "how should the code be organized?" and "what patterns should we use?"

### Sections

**Architecture Style**
- Monolith, microservices, serverless, hybrid?
- Why this choice given the constraints?

**System Overview**
- ASCII diagram or description of major layers/components
- Data flow from client through backend
- How real-time features (if any) work

**Technology Rationale**

For each tech choice (frontend framework, backend runtime, database, etc):
- Why this technology?
- What problems does it solve for this project?
- Known limitations or trade-offs
- Migration path if it becomes wrong

**Layer Architecture**

Depending on your stack:

**Frontend:**
- View/Component organization (pages, reusable components, layout)
- State management (Redux, Context, local state, etc.)
- API client pattern
- Error boundary strategy
- Testing structure

**Backend:**
- Route/Handler organization
- Middleware stack
- Service layer pattern (business logic separation)
- Database access layer
- Error handling and logging
- Testing structure

**Database:**
- Schema design philosophy
- Migration strategy
- Indexing strategy (if applicable)
- Backup/recovery approach

**Deployment & Infrastructure**

- Environments (dev, staging, production)
- How code gets from repo to running
- Environment variables and secrets
- Monitoring and observability
- Log aggregation

**Immutable Decisions**

These are decisions that, if wrong, are expensive to change. Once set:

- Technology choices (can't easily swap React for Vue)
- Data model fundamentals (schema changes are costly)
- Architecture patterns (switching from monolith to microservices is a rewrite)
- Authentication system (changing how users log in affects everything)

**What NOT to Change**

List specific things that should remain consistent:
- Database schema structure
- Core API contracts
- Component naming conventions
- Environment configuration approach
- Authentication flow

If a developer needs to change these, they should flag it for architectural review.

---

## Document 3: PHASES.md

### Purpose
Break the project into buildable work phases. Each phase = 1-3 Claude Code sessions.

### Scope

**Phase 1: Foundation** *(Always First)*
- Project scaffolding (boilerplate, config, tooling)
- Core dependencies installed
- CI/CD pipeline set up (linting, testing, builds)
- Basic project structure in place
- Authentication skeleton (can login but no real features yet)

**Example Phase 1 for a React+Python project:**
- Initialize React app with routing
- Set up Python FastAPI server
- Database migrations and schema
- User model and auth endpoints
- Integration testing framework
- GitHub Actions CI pipeline

**Phase 2, 3, N: Features**

Each feature phase includes:

```
## Phase N: [Feature Name]

**Deliverable:** [What ships after this phase]

**Features:**
- [ ] Feature A
- [ ] Feature B

**Work Items:**
- Backend: [task 1], [task 2]
- Frontend: [task 3], [task 4]
- Testing: [task 5]

**Dependencies:** Must complete Phase M first

**Verification Checklist:**
- [ ] All tests pass
- [ ] Manual testing of flows
- [ ] No console errors
- [ ] Performance acceptable
- [ ] Database migrations applied
- [ ] Environment variables documented

**Time Estimate:** X sessions

**Notes:** Any gotchas, non-obvious implementation details
```

**Final Phase: Polish & Deployment** *(Always Last)*
- Error page design
- Loading states and skeletons
- Performance optimization
- Security audit (inputs, outputs, auth)
- Documentation review
- Staging environment testing
- Production deployment
- Post-launch monitoring setup

### Phase Sizing

Phases are sized for 1-3 Claude Code sessions (~1-3 hours each):

- Too big: Scope creep, hard to verify, risky
- Too small: Constant context switching, inefficient

Aim for tangible, shippable increments.

---

## Document 4: DESIGN_SYSTEM.md

### Purpose
*(Only generated if the project has a UI)*

Codifies the visual and interaction language so all UI is consistent.

### Sections

**Design Philosophy**
- What aesthetic are we going for? (minimal, playful, professional, etc.)
- Why these choices?

**Color Palette**

```
## Colors

### Primary
- Primary Blue: #0066CC (actions, links, primary CTA)
- Primary Blue Dark: #003D99 (hover state)

### Semantic
- Success Green: #10B981
- Warning Yellow: #F59E0B
- Danger Red: #EF4444
- Neutral Gray-50: #F9FAFB
- Neutral Gray-900: #111827

### Text
- Text Primary: #111827 (on light backgrounds)
- Text Secondary: #6B7280
```

**Typography**

```
## Fonts

### Typefaces
- Headings: Inter, sans-serif (weight: 600, 700)
- Body: Inter, sans-serif (weight: 400, 500)

### Scale
- H1: 32px / 1.2 line-height (bold headings)
- H2: 24px / 1.3 line-height
- H3: 20px / 1.4 line-height
- Body: 16px / 1.5 line-height
- Small: 14px / 1.4 line-height
- Caption: 12px / 1.4 line-height
```

**Spacing System**

```
## Spacing

All spacing follows an 8px base unit:

- xs: 4px (small gaps)
- sm: 8px (default spacing)
- md: 16px (section spacing)
- lg: 24px (major sections)
- xl: 32px (page margins)
- 2xl: 48px (between distinct sections)
```

**Component Specifications**

For each major component, document:

```
## Button

**Sizes:**
- Small: 32px height, 12px label, 8px padding
- Medium: 40px height, 14px label, 12px padding
- Large: 48px height, 16px label, 16px padding

**Variants:**
- Primary: Blue background, white text, hover darkens
- Secondary: Gray background, gray text, hover softens
- Ghost: No background, gray text, hover lightens background

**States:**
- Normal: [appearance]
- Hover: [appearance]
- Active/Pressed: [appearance]
- Disabled: 50% opacity, no hover

**Usage:**
- Primary for main CTAs
- Secondary for alternatives
- Ghost for tertiary actions
```

**Grid & Layout**

- Container max-width
- Grid columns (e.g., 12-column grid for responsive breakpoints)
- Responsive breakpoints (mobile, tablet, desktop)

**Interaction Patterns**

- Hover states
- Focus states (keyboard navigation)
- Loading states
- Empty states
- Error states
- Success feedback

**Motion & Transitions**

- Standard duration (e.g., 200ms for UI transitions)
- Easing functions (ease-in-out, ease-out, etc.)
- What should animate (buttons, modals, page transitions)
- What should NOT animate (critical alerts)

**Accessibility**

- Minimum contrast ratios (WCAG AA)
- Focus indicators
- Touch target sizes (minimum 44x44px)
- Alt text strategy
- Keyboard navigation rules

---

## Verification Checklist

After generating specs, I'll verify:

- [ ] All v1 features are documented
- [ ] Each feature has clear success criteria
- [ ] Data models are complete and consistent
- [ ] API routes match data model operations
- [ ] Architecture decisions are justified
- [ ] Immutable decisions are clearly marked
- [ ] Phases are sized appropriately (1-3 sessions each)
- [ ] Phase 1 is foundation only
- [ ] Phase N (final) is polish & deployment
- [ ] All JSON and code samples are valid syntax
- [ ] Design system covers all major components (if UI project)

---

## Quality Standards

Each document should:

- **Be complete enough to code from** — a developer shouldn't have to guess
- **Be concise** — no fluff, no prose that could be a diagram
- **Be consistent** — same terminology, same detail level across documents
- **Be actionable** — organized so developers know what to build and in what order

---

## Ready?

Run `/spec` to generate your specification suite.

The output will be ready for implementation with `/bootstrap`.
