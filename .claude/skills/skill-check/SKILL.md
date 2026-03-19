---
name: "skill-check"
description: "Meta-skill that audits other skills for accuracy, completeness, and relevance. Maintains the skill system."
---

# /skill-check — Skill System Audit

Meta-skill that audits the entire skill system to keep it healthy, accurate, and up-to-date.

## Purpose
Skills drive how work gets done. If skills become stale, outdated, or incomplete, the entire system degrades. This skill periodically audits and fixes the skill system itself.

## When to Run /skill-check
- ✅ End of each phase (via `/phase-review`)
- ✅ Quarterly project audits
- ✅ When you notice skills are outdated
- ✅ Before starting a new major feature
- ✅ After discovering a repeated pattern (might need a new skill)

## Workflow

### 1. INVENTORY ALL SKILLS
Scan `.claude/skills/` directory:
```bash
find .claude/skills/ -name "SKILL.md"
```

For each skill file, record:
- Skill name (from YAML frontmatter)
- Skill path (e.g., `.claude/skills/dev/SKILL.md`)
- Description

### 2. INVENTORY ALL AGENTS & RULES
Check for:
- Agents referenced in skills (e.g., `@code-reviewer`)
- Global rules in `.claude/RULES.md`
- Stack-specific skills (if any)

### 3. VERIFY FILE PATHS MENTIONED IN SKILLS
For each skill, extract file paths mentioned:
- Example: skill mentions "CLAUDE.md", "PHASES.md", "transfer-context.json"
- Verify these files exist

Result: Path validation report
```
✓ CLAUDE.md exists
✓ PHASES.md exists
✓ transfer-context.json exists (in .claude/)
✓ .claude/RULES.md exists
```

### 4. VERIFY CODE PATTERNS MATCH CODEBASE
For each skill, identify code examples or patterns mentioned:
- Example: `/tdd` skill mentions "test framework" and "assertions"
- Example: `/dev` skill mentions "commit messages"
- Verify these patterns are actually used in the current codebase

Result: Pattern validation
```
✓ Commits use descriptive messages
✓ Tests use assertions correctly
✓ YAML frontmatter is in all skills
```

If a pattern is missing:
- Update the skill to match reality
- Or ask user: "Should we start using this pattern?"

### 5. CHECK FOR REPEATED PATTERNS
Review codebase and git history:
```bash
git log --oneline -50 | grep -E "PATTERN|FIX|FEATURE"
```

Look for:
- Repeated problems (e.g., "fixed pagination bug 3 times")
- Repeated tasks (e.g., "add API endpoint" done 5 times)
- Repeated patterns (e.g., database migrations always use X)

For each repeated pattern:
- Is there a skill for this? If not, suggest a new skill
- Example: "I see 5 commits about API endpoints. Should we create `/new-endpoint` skill?"

### 6. RATE EACH SKILL

For each skill, rate its status:

| Status | Meaning | Action |
|--------|---------|--------|
| **Current** | Up-to-date, accurate, useful | Keep as-is |
| **Stale** | Outdated info, no longer relevant | Update or archive |
| **Incomplete** | Missing steps or info | Complete it |
| **Redundant** | Overlaps with another skill | Merge or delete |
| **Missing** | Pattern repeats but no skill | Create new skill |

Example rating:
```
✓ CURRENT: /dev (master workflow, actively used)
✓ CURRENT: /tdd (TDD cycle accurate)
⚠ STALE: /learn (mentions old CLAUDE.md location)
✓ CURRENT: /debug (structured debugging still accurate)
⚠ INCOMPLETE: /new-feature (too generic, needs stack-specific versions)
⚠ REDUNDANT: /new-api (overlaps with /dev, could be /dev subsection)
⊕ MISSING: /db-migration (pattern repeated 6x, no skill yet)
```

### 7. FIX STALE/INCOMPLETE SKILLS
For each skill marked stale or incomplete:
- Update it
- Commit: `git commit -m "SKILL-CHECK: update [skill-name]"`

Example fixes:
```
STALE /learn:
  - Update CLAUDE.md path references
  - Commit: "SKILL-CHECK: update /learn paths"

INCOMPLETE /new-feature:
  - Add note: "This is generic. Run /bootstrap to get stack-specific skills"
  - Commit: "SKILL-CHECK: enhance /new-feature with bootstrap note"
```

### 8. SUGGEST NEW SKILLS
For each repeated pattern identified:
- What is the pattern?
- How many times has it occurred?
- What skill would solve it?

Present to user:
```
SUGGESTED NEW SKILLS:

Pattern: API endpoint creation (6 commits matching /api/*)
Current workaround: Use /dev skill, manually follow pattern
Suggested: Create /new-endpoint skill
  - Pre-filled boilerplate for new endpoint
  - Validation template
  - Test template
  - Auto-adds route registration

Pattern: Database schema changes (4 commits, 3 were migrations)
Current workaround: Manual migration writing
Suggested: Create /db-migration skill
  - Migration template
  - Rollback template
  - Naming convention
  - Auto-tracks migration version

Should I create these?
```

If user approves, create new skill files:
```bash
mkdir .claude/skills/new-endpoint
cat > .claude/skills/new-endpoint/SKILL.md << 'EOF'
---
name: "new-endpoint"
description: "Create a new API endpoint with validation and tests."
---
# /new-endpoint
[template]
EOF
```

### 9. PRESENT HEALTH REPORT
Show user a summary of skill system health:

```
╔════════════════════════════════════════════════╗
║            SKILL SYSTEM HEALTH REPORT          ║
╚════════════════════════════════════════════════╝

SKILL INVENTORY
Total skills: 8
Phase B skills: 8 (/dev, /wrap-up, /tdd, /debug, /learn, /phase-review, /skill-check, /new-feature)
Stack-specific skills: 0 (would be created by /bootstrap)

SKILL STATUS BREAKDOWN
✓ Current:    6 (/dev, /wrap-up, /tdd, /debug, /learn, /phase-review)
⚠ Incomplete: 1 (/new-feature — needs stack-specific versions)
✓ Current:    1 (/skill-check)

STATUS SUMMARY
□ No stale skills
□ No redundant skills
✓ All core paths exist
✓ Code patterns match codebase

PATTERNS DETECTED
Pattern: Database queries (5 commits)
  → Already have best practices in /learn
  → Consider /db-query skill if queries become complex

Pattern: Test setup (3 commits, all fixable by /tdd)
  → /tdd skill covers this, no new skill needed

RECOMMENDATIONS
1. Current system is healthy
2. After /bootstrap runs, expect stack-specific skills
3. Monitor for: repeated /db-* patterns (might justify /db-query skill)
4. Consider creating /new-endpoint if API gets complex

LAST CHECKED
  Date: 2026-03-19
  Duration: 5 minutes
  Issues found: 0
  Skills needing updates: 0
```

### 10. UPDATE SKILL REGISTRY (IF EXISTS)
If there's a `.claude/skills-registry.json`, update it:

```json
{
  "lastChecked": "2026-03-19T15:45:00Z",
  "totalSkills": 8,
  "healthStatus": "healthy",
  "skillsCurrentCount": 7,
  "skillsStaleCount": 0,
  "skillsIncompleteCount": 1,
  "suggestedNewSkills": [],
  "skills": [
    {
      "name": "dev",
      "status": "current",
      "usageCount": 15,
      "lastModified": "2026-03-19"
    },
    ...
  ]
}
```

### 11. COMMIT AUDIT RESULTS
```bash
git add .claude/
git commit -m "SKILL-CHECK: health audit complete, 7 current, 0 stale, system healthy"
```

## Example Audit Session

```
SKILL-CHECK: Running audit...

=== INVENTORY ===
Found 8 skills: /dev, /wrap-up, /tdd, /debug, /learn, /phase-review, /skill-check, /new-feature

=== PATH VALIDATION ===
✓ All file paths mentioned in skills exist
✓ CLAUDE.md in place
✓ PHASES.md in place
✓ transfer-context.json in place

=== PATTERN MATCHING ===
✓ Commit messages use descriptive format
✓ YAML frontmatter in all skills
✓ Test assertions follow convention

=== REPEATED PATTERNS ===
Scanned last 50 commits:
  - API endpoints (5 commits): well-covered by /dev
  - Database queries (4 commits): documented in /learn lessons
  - Test setup (3 commits): /tdd skill covers
  - Debugging (2 commits): /debug skill covers

=== SKILL RATINGS ===
✓ /dev: CURRENT — master workflow, well-maintained
✓ /wrap-up: CURRENT — session end ritual, accurate
✓ /tdd: CURRENT — TDD cycle clear and useful
✓ /debug: CURRENT — structured debugging, effective
✓ /learn: CURRENT — lesson capture working
✓ /phase-review: CURRENT — phase audits working
✓ /skill-check: CURRENT — this skill, self-referential
⚠ /new-feature: INCOMPLETE — too generic, needs note about /bootstrap

=== FIXES NEEDED ===
1. Update /new-feature with bootstrap note

=== NEW SKILLS SUGGESTED ===
None at this time. Current patterns are covered.

=== AUDIT COMPLETE ===
Status: HEALTHY
Skills current: 7/8
Skills needing work: 1
Recommended actions: Update /new-feature note

Ready to move to next phase ✓
```

## Checklist
- [ ] Inventoried all skills
- [ ] Listed agents and rules
- [ ] Verified all mentioned file paths exist
- [ ] Checked code patterns match codebase
- [ ] Scanned for repeated patterns
- [ ] Rated each skill (current/stale/incomplete/redundant/missing)
- [ ] Fixed stale/incomplete skills
- [ ] Suggested new skills to user
- [ ] Generated health report
- [ ] Committed audit results

## Key Principles
1. **Meta-awareness**: This skill audits skills themselves
2. **Health matters**: Bad skills lead to bad work
3. **Pattern recognition**: Repeated patterns suggest new skills
4. **Keep current**: Update skills as codebase evolves
5. **Transparency**: Show user the audit, get feedback
6. **Continuous improvement**: Skills should get better over time

## Integration with Other Skills
- **/phase-review**: Calls /skill-check at end of phase
- **/bootstrap**: Creates stack-specific skills after project setup
- All other skills: /skill-check audits them
