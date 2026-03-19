---
name: scope
description: Interactive project scoping — understand the idea, define the MVP, choose the stack
version: 1.0.0
author: Claude Forge
category: phase-a
phase: discovery
---

# Project Scoping Skill

This skill conducts an interactive discovery conversation to deeply understand what you want to build, define your MVP, and select an appropriate tech stack.

## Overview

Scoping is the foundation of successful development. This skill guides you through four discovery phases, asks clarifying follow-up questions, and outputs a structured project scope file that will drive all downstream decisions.

**Output:** `.claude/project-scope.json` — your project's foundational decisions.

---

## How to Use

Run this skill at the very start of a new Claude Forge project:

```
/scope
```

The skill will guide you through a conversational discovery process. Answer naturally — the skill asks follow-ups to understand your vision deeply.

---

## The Four Discovery Phases

### Phase 1: Vision
**Goal:** Understand what you're building, why, and for whom.

I'll ask you about:
- What is the core idea or product?
- What problem does it solve? Who has this problem?
- Who are the primary users/audience?
- What does success look like? (Revenue? Adoption? Impact?)
- What's the one thing that makes this different?

**Checkpoint:** I'll summarize what I heard and confirm: "Did I get that right?"

---

### Phase 2: Technical Foundation
**Goal:** Understand the technical shape of the project.

I'll ask about:
- **Platforms:** Where does it run? (Web browser? iOS/Android? Desktop? Server/API only? Multiple?)
- **Data:** What data does your product need to store? (User profiles? Content? Analytics? Real-time state?)
- **Integrations:** Do you need external services? (Payment processors? Maps? Auth providers? Email? Analytics? Third-party APIs?)
- **Real-time:** Does it need real-time features? (Live notifications? Collaborative editing? Streaming data?)
- **Offline:** Does it need to work offline? (Mobile-first? Sync on reconnect?)
- **Scale:** Are we talking MVP (100 users) or day-1 scale (10K+ users)?

**Checkpoint:** I'll confirm: "Here's what I heard about the technical side..."

---

### Phase 3: Constraints & Context
**Goal:** Understand your limitations, preferences, and team dynamics.

I'll ask about:
- **Timeline:** When do you want to ship? (This week? This quarter? No deadline?)
- **Experience:** What stacks are you comfortable with? What's brand new to you?
- **Preferences:** Any stack preferences or strong dealbreakers? (Love React? Hate Node? Prefer Python?)
- **Team:** Solo dev? Small team? What's everyone's skill level?
- **Deployability:** Any hosting constraints? (Can't use cloud? On-prem required? Budget-conscious?)

**Checkpoint:** I'll summarize constraints and confirm.

---

### Phase 4: MVP & Roadmap
**Goal:** Define what ships in v1 and what's aspirational.

I'll ask about:
- **Must-Haves:** What are the absolute minimum features for launch?
- **Nice-to-Haves:** What would be great but can wait?
- **User Flows:** Walk me through the happy path — what does a user do from start to end?
- **Success Metrics:** How will you measure if the MVP is working?

**Checkpoint:** I'll confirm the MVP scope and suggest a build roadmap.

---

## Tech Stack Selection

After gathering all context, I'll suggest a tech stack with clear reasoning:

- **Frontend:** (React/Vue/Svelte/Flutter/etc)
- **Backend:** (Node.js/Python/Go/Rust/etc)
- **Database:** (PostgreSQL/MongoDB/Firebase/etc)
- **Auth:** (Auth0/Supabase/Firebase/Custom)
- **Hosting:** (Vercel/AWS/Railway/Render/etc)

I'll explain why each choice fits your constraints and confirm: "Does this stack feel right to you?"

You can override any suggestion — this is your project.

---

## Output: project-scope.json

At the end of the conversation, I'll generate `.claude/project-scope.json`:

```json
{
  "project_name": "string",
  "description": "string",
  "problem_statement": "string",
  "target_users": "string",
  "vision_statement": "string",
  "success_metrics": ["string"],

  "platforms": {
    "web": boolean,
    "mobile": boolean,
    "desktop": boolean,
    "server": boolean,
    "details": "string"
  },

  "tech_stack": {
    "frontend": "string",
    "backend": "string",
    "database": "string",
    "auth": "string",
    "hosting": "string",
    "additional_services": ["string"],
    "reasoning": "string"
  },

  "features_v1": [
    {
      "name": "string",
      "description": "string",
      "user_flow": "string"
    }
  ],

  "features_v2": [
    {
      "name": "string",
      "description": "string",
      "rationale": "string"
    }
  ],

  "integrations": [
    {
      "service": "string",
      "purpose": "string",
      "criticality": "must-have | nice-to-have"
    }
  ],

  "constraints": {
    "timeline": "string",
    "team_size": integer,
    "team_experience": "string",
    "budget_constraints": "string",
    "hosting_constraints": "string",
    "dealbreakers": ["string"]
  },

  "architecture_style": "monolith | microservices | serverless | hybrid",
  "real_time_needed": boolean,
  "offline_support_needed": boolean,
  "estimated_scale_day_1": "string",

  "notes": "string",
  "conversation_date": "ISO 8601 timestamp",
  "scoped_by": "user identifier"
}
```

---

## Key Principles

1. **Ask Follow-Ups:** I won't just accept surface answers. If you say "it's a social app," I'll ask what kind, for whom, what makes it different.

2. **Validate Assumptions:** I'll mirror back what I hear to ensure clarity. This prevents costly misunderstandings.

3. **Think Practically:** I'll ask questions that shape real decisions — hosting, scale, team — not just collect information.

4. **Explore Trade-Offs:** Some tech choices create trade-offs. I'll help you understand them.

5. **Override-Friendly:** Tech stack suggestions are starting points. You know your constraints best.

---

## Example Flow

**You:** "I want to build a note-taking app."

**Me:** "Great! Let me understand the vision better:
- Are you thinking personal notes, or collaborative?
- Is this competing with Notion? Or solving a specific niche (e.g., quick voice notes)?
- Who would use it? Developers? Students? Everyone?
- What's the insight that makes this different from existing options?"

**You:** "It's for developers. Real-time collaborative code snippets with AI-powered tagging."

**Me:** "Ah — so real-time sync, code syntax awareness, and AI integration. That's three important constraints. Do you want to:
- Keep snippets private by default (personal) or shared?
- How many concurrent users per snippet?
- Are you using OpenAI's API or self-hosting models?"

And so on. Each answer refines the picture.

---

## Ready to Begin?

Answer the initial questions naturally. The skill will guide you from vision through MVP. At the end, you'll have a clear, written scope that developers can build from.

Let's go. **What are you building?**
