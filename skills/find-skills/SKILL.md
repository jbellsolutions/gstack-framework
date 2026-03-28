---
name: gf-find-skills
preamble-tier: 2
version: 1.0.0
description: |
  Analyze a repo to identify where skills should exist across the full development lifecycle.
  Maps workflow stages, detects gaps, and proposes skills with names, descriptions, and purposes.
  Use when asked to "find skills", "what skills does this project need", or "where should we add skills".
  Proactively suggest when a project has few or no skills and the user wants to expand coverage.
  For creating the actual skill files after approval, use /gf-create-skills instead.
allowed-tools:
  - Bash
  - Read
  - Glob
  - Grep
  - AskUserQuestion
  - WebSearch
---

## Preamble (run first)

```bash
# Session tracking
mkdir -p ~/.gstack/sessions
touch ~/.gstack/sessions/"$PPID"
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null || pwd)
echo "BRANCH: $_BRANCH"
echo "ROOT: $_ROOT"
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

# Find Skills

Analyze a repository's full development lifecycle and identify where skills are missing, weak, or would deliver high value. Output a prioritized list of proposed skills for user approval.

## Iron Law

**NO SKILL PROPOSALS WITHOUT EVIDENCE FROM THE ACTUAL CODEBASE.**

Every proposed skill must cite a concrete file, pattern, or workflow gap found in the repo. No hypothetical skills based on "best practices" alone.

---

## Phase 1: Load Context

Read the analysis report from /gf-analyze if it exists. Understand the project before proposing anything.

1. Check for existing analysis:
   ```bash
   find "$_ROOT" -name "gf-analysis-report.*" -o -name "analysis-report.*" | head -5
   ```
2. Read the report if found. Extract: tech stack, project type, existing skills, key directories.
3. If no report exists, do a quick scan:
   - `ls` the root, `src/`, `lib/`, `app/` directories
   - Read `package.json`, `Cargo.toml`, `pyproject.toml`, or equivalent
   - Check for existing `.claude/skills/`, `.claude/commands/`, or `skills/` directories
4. Read any existing SKILL.md files to understand current coverage.

**Output:** One-paragraph summary of the project and what's already covered.

---

## Phase 2: Map Workflow Stages

Check coverage across 8 universal stages plus domain-specific stages.

### Universal Stages

| # | Stage | Example G-Stack Skills | What to Look For |
|---|-------|----------------------|------------------|
| 1 | Brainstorming/Ideation | /office-hours | Design docs, RFCs, ADRs, brainstorm notes |
| 2 | Planning/Architecture | /plan-ceo-review, /plan-eng-review | Architecture files, planning docs, specs |
| 3 | Design | /design-consultation, /plan-design-review | UI components, design tokens, style guides |
| 4 | Building/Implementation | (custom to domain) | Source code, build configs, generators |
| 5 | Testing/QA | /qa, /investigate | Test files, test configs, CI test steps |
| 6 | Code Review | /review | PR templates, linting configs, review checklists |
| 7 | Shipping/Deployment | /ship, /land-and-deploy | Deploy scripts, CI/CD configs, release workflows |
| 8 | Monitoring/Operations | /canary, /benchmark | Monitoring configs, alerting, health checks |

### Domain-Specific Stages

Based on the project type, add extra stages:

- **Web app:** Auth flow, form handling, SEO, accessibility, state management
- **API:** Endpoint scaffolding, documentation generation, rate limiting, versioning
- **CLI tool:** Command testing, help generation, shell completion, man pages
- **Content/media:** Content pipeline, quality checking, asset optimization
- **Data:** Pipeline orchestration, validation, migration, schema evolution
- **Mobile:** Device testing, app store prep, deep linking

For each stage, mark: COVERED (skill exists), PARTIAL (some coverage), or MISSING.

---

## Phase 3: Gap Analysis

For each MISSING or PARTIAL stage:

1. Search the codebase for related files:
   ```bash
   # Example: checking for deploy-related files
   find "$_ROOT" -name "*.yml" -o -name "*.yaml" | xargs grep -l "deploy\|release\|publish" 2>/dev/null
   ```
2. Identify what a skill would actually need to do by reading those files.
3. Note the specific pain point: What goes wrong without this skill? What takes too long?

**Only stop for:**
- Cannot determine project type after reading configs
- No source code found in the repo

**Never stop for:**
- Missing a single stage (some stages are legitimately unnecessary)
- Unclear tech stack details (propose based on what you can see)

---

## Phase 4: Domain-Specific Skills

Based on what Phase 1 revealed about the project type, identify unique skill opportunities.

### Web App Skills
| Opportunity | Evidence to Find | Skill Purpose |
|-------------|-----------------|---------------|
| Form validation | Form components, validation libs | Standardize validation patterns |
| Auth flow | Auth middleware, login pages | Automate auth testing and setup |
| SEO | Meta tags, sitemap, robots.txt | Audit and fix SEO issues |
| Accessibility | ARIA attributes, a11y tests | Run accessibility checks |

### API Skills
| Opportunity | Evidence to Find | Skill Purpose |
|-------------|-----------------|---------------|
| Endpoint testing | Route files, controllers | Generate and run endpoint tests |
| API docs | OpenAPI specs, doc comments | Generate/update API documentation |
| Rate limiting | Middleware, config files | Configure and test rate limits |

### CLI Tool Skills
| Opportunity | Evidence to Find | Skill Purpose |
|-------------|-----------------|---------------|
| Command testing | Command files, argument parsers | Test all commands systematically |
| Help generation | Help strings, usage docs | Keep help text in sync with code |

### Content/Media Skills
| Opportunity | Evidence to Find | Skill Purpose |
|-------------|-----------------|---------------|
| Content pipeline | Content directories, processors | Automate content processing |
| Quality checker | Linters, validators | Check content quality standards |

### Data Skills
| Opportunity | Evidence to Find | Skill Purpose |
|-------------|-----------------|---------------|
| Pipeline | ETL scripts, data processors | Orchestrate data pipeline runs |
| Validation | Schema files, validators | Validate data integrity |
| Migration | Migration files, schema changes | Generate and run migrations safely |

Search the codebase for evidence before proposing any domain skill.

---

## Phase 5: Prioritize

Score each proposed skill on two axes:

- **Impact (1-10):** How much time/pain does this save? How often would it run?
- **Effort (1-10):** How complex is this skill to build? 1 = simple wrapper, 10 = multi-phase agent.

Calculate **priority ratio** = Impact / Effort. Sort descending.

Assign priority labels:
- **P0:** Ratio > 3.0, used daily
- **P1:** Ratio > 2.0, used weekly
- **P2:** Ratio > 1.0, used regularly
- **P3:** Ratio > 0.5, nice to have
- **P4:** Ratio <= 0.5, low priority

---

## Phase 6: Propose

For each recommended skill, output this format:

```
### [skill-name-slug]
- **Description:** One line with trigger phrases
- **What it does:** 2-3 sentences on the actual behavior
- **Workflow stage:** Which of the 8+ stages it covers
- **Priority:** P0-P4 (Impact: X/10, Effort: X/10, Ratio: X.X)
- **Complexity:** simple | medium | complex
- **Evidence:** The specific files/patterns that justify this skill
```

Group by workflow stage. Put P0 and P1 skills first within each group.

---

## Phase 7: User Approval

Present the top 5-10 skills via AskUserQuestion.

Format:

1. **Re-ground:** "Analyzing [project] on [branch]. Found [N] skill opportunities across [M] workflow stages."
2. **Simplify:** "Here are the highest-value skills I found. Pick which ones to create."
3. **Recommend:** RECOMMENDATION: Start with [top 3] because [reasons]. Completeness: X/10
4. **Options:**
   - A) Create top 3 (P0 only) ... effort: AI ~30 min
   - B) Create top 5 (P0 + P1) ... effort: AI ~1 hour
   - C) Create all proposed ... effort: AI ~2 hours
   - D) Let me pick specific ones from the list
   - E) Skip skill creation, just save the analysis

After approval, hand off to /gf-create-skills with the selected list.

---

## Phase 8: Verification

**IRON LAW: NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE.**

1. Verify every proposed skill cites at least one real file or pattern from the codebase.
2. Verify no duplicate skills (different name, same purpose).
3. Verify priority scores are consistent (a P0 should clearly matter more than a P2).

**Rationalization prevention:**
- "This is a best practice" -> Show the file that needs it
- "Every project needs this" -> Prove it from this codebase
- "I'm confident in the rankings" -> Re-check the impact/effort math

---

## Phase 9: Report

```
REPORT
================================
Task:            Find skills for [project]
Project type:    [type detected]
Stages checked:  [N] universal + [M] domain-specific
Coverage:        [X] covered, [Y] partial, [Z] missing
Skills proposed: [total] (P0: [n], P1: [n], P2: [n], P3+: [n])
Top 3:           [skill1], [skill2], [skill3]
User decision:   [what they chose]
Status:          DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
================================
```

---

## Completion Status

- **DONE** -- All stages analyzed, skills proposed, user approved selections
- **DONE_WITH_CONCERNS** -- Completed but project type unclear or coverage uncertain
- **BLOCKED** -- Cannot determine project structure. What was tried. What to do next.
- **NEEDS_CONTEXT** -- Missing analysis report. Run /gf-analyze first.

### Escalation
- 3 failed attempts to identify project type -> STOP and escalate
- Repo has no source code -> STOP and escalate
- Bad proposals are worse than no proposals

---

## Important Rules

- Never propose skills without evidence from the codebase
- Never propose more than 15 skills (focus beats breadth)
- Always check for existing skills before proposing duplicates
- Always present proposals sorted by priority ratio
- 3-strike rule: if 3 proposed skills have no codebase evidence, stop and re-analyze
