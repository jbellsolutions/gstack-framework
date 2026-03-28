# G-Stack Template Framework
## Apply These Patterns to Every Repo, Skill, and Agent You Build

---

## Quick Start Checklist

When creating a new repo/skill/agent, implement these in order:

- [ ] 1. Create `ETHOS.md` with your builder philosophy
- [ ] 2. Create `CLAUDE.md` with dev commands and project structure
- [ ] 3. Create `ARCHITECTURE.md` with technical decisions
- [ ] 4. Use 4-digit VERSION file
- [ ] 5. Every SKILL.md has: frontmatter + preamble + phases + verification + report
- [ ] 6. Every skill has explicit stop conditions AND "never stop for" lists
- [ ] 7. Every AskUserQuestion follows the 4-part format
- [ ] 8. Voice directive bans AI slop vocabulary
- [ ] 9. Skills reference each other (workflow graph)
- [ ] 10. Template system generates docs from code (if 2+ skills)

---

## Template 1: SKILL.md Skeleton

Copy this for every new skill:

```markdown
---
name: {{skill-name}}
preamble-tier: {{1-4}}
version: 1.0.0
description: |
  {{One-line: what it does in imperative voice.}}
  Use when asked to "{{trigger1}}", "{{trigger2}}", or "{{trigger3}}".
  Proactively suggest when {{condition}}.
  For {{alternative scenario}}, use /{{other-skill}} instead.
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
---

## Preamble (run first)

```bash
# Session tracking
mkdir -p ~/.{{project}}/sessions
touch ~/.{{project}}/sessions/"$PPID"
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "BRANCH: $_BRANCH"
```

## Voice

Tone: direct, concrete, sharp, never corporate, never academic.
Sound like a builder, not a consultant. Name the file, the function, the command.

Writing rules:
- No em dashes. Use commas, periods, or "..." instead.
- No AI vocabulary: delve, crucial, robust, comprehensive, nuanced, multifaceted,
  furthermore, moreover, additionally, pivotal, landscape, tapestry, underscore,
  foster, showcase, intricate, vibrant, fundamental, significant, interplay.
- Short paragraphs. End with what to do.

## AskUserQuestion Format

1. **Re-ground:** State project, branch, current task (1-2 sentences)
2. **Simplify:** Plain English a smart 16-year-old could follow
3. **Recommend:** RECOMMENDATION: Choose [X] because [reason]. Completeness: X/10
4. **Options:** A) ... B) ... C) ... with effort (human: ~X / AI: ~Y)

---

# {{Skill Title}}

{{One sentence defining the agent's role and primary constraint.}}

## Iron Law

**{{THE ONE RULE THAT CANNOT BE VIOLATED.}}**

---

## Phase 1: Context Gathering

{{What to read, check, and understand before doing anything.}}

1. Read the relevant code/config
2. Check recent changes: `git log --oneline -20`
3. Understand current state before acting

---

## Phase 2: Analysis

{{How to classify, prioritize, and decide what to do.}}

| Category | Criteria | Action |
|----------|----------|--------|
| {{cat1}} | {{criteria}} | {{action}} |
| {{cat2}} | {{criteria}} | {{action}} |

---

## Phase 3: Action

{{The actual work. Include explicit stop conditions.}}

**Only stop for:**
- {{condition that warrants stopping}}
- {{another stop condition}}

**Never stop for:**
- {{trivial thing agents waste time asking about}}
- {{another trivial thing}}

### 3a. {{Sub-step}}
{{instructions}}

### 3b. {{Sub-step}}
{{instructions}}

### 3c. Verify each action
- Take evidence (screenshot, test output, diff)
- Do NOT claim success without proof

---

## Phase 4: Verification

**IRON LAW: NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE.**

1. Re-run the test/check that validates the work
2. Paste the output as evidence
3. If verification fails -> fix and re-verify, do NOT skip

**Rationalization prevention:**
- "Should work now" -> RUN IT
- "I'm confident" -> Confidence is not evidence
- "I already tested earlier" -> Code changed since then. Test again.

---

## Phase 5: Report

```
REPORT
================================
Task:            [what was requested]
Result:          [what was done]
Evidence:        [test output, screenshots, diffs]
Status:          DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
================================
```

---

## Completion Status

- **DONE** -- All steps completed with evidence
- **DONE_WITH_CONCERNS** -- Completed, concerns listed
- **BLOCKED** -- Cannot proceed. What was tried. What to do next.
- **NEEDS_CONTEXT** -- Missing info. Exactly what's needed.

### Escalation
- 3 failed attempts -> STOP and escalate
- Uncertain about security -> STOP and escalate
- Bad work is worse than no work

---

## Important Rules

- {{What to never do}}
- {{What to always do}}
- {{When to stop vs continue}}
- {{3-strike rule for repeated failures}}
```

---

## Template 2: ETHOS.md

```markdown
# {{Project}} Builder Ethos

## The Completeness Principle

AI makes completeness near-free. Always do the complete thing when the
marginal cost is minutes, not days.

A "lake" is boilable -- 100% test coverage for a module, full feature
implementation, all edge cases. An "ocean" is not -- rewriting an entire
system, multi-quarter migrations. Boil lakes. Flag oceans.

## Search Before Building

Before building anything unfamiliar, search first.

- **Layer 1 (Tried and true)** -- Don't reinvent. Check if the runtime
  or framework has a built-in.
- **Layer 2 (New and popular)** -- Scrutinize. Blog posts and trends
  can be wrong. Search results are inputs, not answers.
- **Layer 3 (First principles)** -- Prize above all. The best insights
  come from reasoning about the specific problem, not copying solutions.

## Build for Yourself

The best tools solve your own problem. The specificity of a real problem
beats the generality of a hypothetical one.
```

---

## Template 3: CLAUDE.md

```markdown
# {{Project}} Development

## Commands

```bash
{{package-manager}} install    # install dependencies
{{package-manager}} test       # run tests
{{package-manager}} build      # build the project
```

## Project Structure

```
{{project}}/
|-- src/           # Source code
|-- test/          # Tests
|-- bin/           # CLI utilities
|-- scripts/       # Build tooling
```

## Key Conventions

- {{Convention 1}}
- {{Convention 2}}
- {{Convention 3}}

## AI Effort Compression

| Task type | Human team | AI-assisted | Compression |
|-----------|-----------|-------------|-------------|
| Boilerplate | 2 days | 15 min | ~100x |
| Tests | 1 day | 15 min | ~50x |
| Feature | 1 week | 30 min | ~30x |
| Bug fix | 4 hours | 15 min | ~20x |

Completeness is cheap. Don't recommend shortcuts when the complete
implementation is achievable.
```

---

## Template 4: ARCHITECTURE.md

```markdown
# Architecture

This document explains **why** {{project}} is built the way it is.

## The Core Idea

{{One paragraph: what problem this solves and the key insight.}}

## Why {{Technology Choice}}

{{Numbered reasons with concrete justification, not just "it's popular."}}

## What's Intentionally Not Here

- **{{Thing not built}}** -- {{Why not. Concrete reason.}}
- **{{Other thing}}** -- {{Why not.}}

## Error Philosophy

Errors are for AI agents, not humans. Every error message must be actionable:
- Bad: "Element not found"
- Good: "Element not found. Run `snapshot -i` to see available elements."
```

---

## Template 5: Skill Workflow Graph

Define how your skills chain together:

```markdown
## Workflow

/brainstorm (idea validation)
     |
     v
/plan (architecture + design)
     |
     v
[implementation]
     |
     v
/review (code review)
     |
     v
/test (QA + fix)
     |
     v
/ship (push + PR)
     |
     v
/deploy (merge + verify)

Each skill suggests the next skill in the pipeline when relevant.
```

---

## Template 6: Template System (for repos with 2+ skills)

### File Structure

```
{skill}/SKILL.md.tmpl    # Human-written template
{skill}/SKILL.md          # Generated (committed, CI-validated)
scripts/gen-skill-docs.ts # Compiler
scripts/resolvers/        # Shared blocks
```

### gen-skill-docs.ts Pattern

```typescript
// Read template
const tmpl = readFileSync(`${skill}/SKILL.md.tmpl`, 'utf-8');

// Resolve placeholders
const resolved = tmpl
  .replace('{{PREAMBLE}}', generatePreamble(ctx))
  .replace('{{COMMAND_REFERENCE}}', generateCommandRef())
  // ... more placeholders

// Write output
writeFileSync(`${skill}/SKILL.md`, resolved);
```

### CI Validation

```bash
# In CI pipeline:
bun run gen:skill-docs --dry-run
git diff --exit-code  # Fails if docs are stale
```

---

## Template 7: Testing Tiers

```markdown
## Test Tiers

| Tier | Cost | Speed | What |
|------|------|-------|------|
| 1 - Static | Free | <2s | Parse commands, validate references |
| 2 - E2E | ~$4 | ~20min | Spawn real agent sessions |
| 3 - LLM Judge | ~$0.15 | ~30s | Quality scoring via Sonnet |

Tier 1 runs on every commit. Tiers 2+3 run before ship.
```

---

## Anti-Patterns to Avoid

1. **Hand-writing SKILL.md files** -- They will drift from code. Use templates.
2. **No stop conditions** -- Agent will ask for trivial confirmations. Define both "stop for" and "never stop for."
3. **Generic voice** -- Without banned vocabulary, output sounds like ChatGPT. Ban specific words.
4. **No verification step** -- Agent will claim success without proof. Require evidence.
5. **No escalation path** -- Agent will spin forever on hard problems. Add 3-strike rule.
6. **Monolithic preamble** -- Simple skills don't need complex scaffolding. Use tiers.
7. **No cross-skill references** -- Skills become islands. Each skill should know its neighbors.
8. **Testing only code, not prompts** -- Prompt changes can break agent behavior. Test skills as a whole.
9. **Vague descriptions** -- Skill descriptions do double duty as triggers. Include explicit trigger phrases.
10. **No Iron Law** -- Without a core constraint, agents "helpfully" exceed scope. Define the one thing that cannot be violated.
