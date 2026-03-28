# G-Stack Deep Analysis: Why It Works & How to Templatize It

**Date:** 2026-03-27
**Repo:** https://github.com/garrytan/gstack
**Author of analysis:** Claude (requested by user)

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Architecture Overview](#2-architecture-overview)
3. [The 7 Pillars: Why G-Stack Works](#3-the-7-pillars)
4. [SKILL.md Anatomy](#4-skillmd-anatomy)
5. [The Template System](#5-the-template-system)
6. [The Preamble Architecture](#6-the-preamble-architecture)
7. [Prompt Engineering Patterns](#7-prompt-engineering-patterns)
8. [The Voice System](#8-the-voice-system)
9. [Agent Orchestration](#9-agent-orchestration)
10. [The Browse Daemon](#10-the-browse-daemon)
11. [Testing & Eval System](#11-testing--eval-system)
12. [Distribution & Installation](#12-distribution--installation)
13. [Extracted Templates](#13-extracted-templates)
14. [Applying This to Your Own Repos](#14-applying-this-to-your-own-repos)

---

## 1. Executive Summary

G-Stack is not just a collection of skills. It is a **complete AI engineering operating system** built on one core insight:

> An AI agent is only as good as the structure you give it. The structure must be opinionated, generated from source code, and tested like software.

What makes it exceptional:

1. **Skills are generated, not hand-written** - SKILL.md files are compiled from `.tmpl` templates + source code metadata. Docs never drift from code.
2. **Every skill shares a common preamble** - Session tracking, upgrade checks, telemetry, voice directives, and the "Completeness Principle" are injected into every skill automatically.
3. **Skills are tiered by complexity** - 4 tiers control how much scaffolding each skill gets (T1=lightweight browse, T4=full ship/QA workflow).
4. **Phase-based workflows with hard gates** - Every complex skill follows a numbered phase structure with explicit stop conditions and escalation paths.
5. **The browse daemon gives persistent state** - A long-lived Chromium process means sub-second browser commands with cookies and sessions that persist.
6. **Multi-host support** - Same skills work across Claude Code, Codex, Kiro, and Gemini via generated host-specific SKILL.md variants.
7. **E2E evals test skills as a whole** - Skills are tested by spawning real Claude sessions and checking outputs, not just unit-testing the code.

---

## 2. Architecture Overview

```
gstack/
|
|-- CORE PHILOSOPHY
|   |-- ETHOS.md          # "Boil the Lake" + "Search Before Building"
|   |-- ARCHITECTURE.md   # Technical decisions and why
|   |-- DESIGN.md          # Design system
|
|-- SKILL SYSTEM (the heart)
|   |-- {skill}/SKILL.md.tmpl    # Human-written template (edit this)
|   |-- {skill}/SKILL.md         # Generated output (don't edit)
|   |-- scripts/gen-skill-docs.ts # Template -> SKILL.md compiler
|   |-- scripts/resolvers/       # Shared template blocks
|   |   |-- preamble.ts          # Universal preamble (tiers 1-4)
|   |   |-- browse.ts            # Browse setup block
|   |   |-- testing.ts           # Test bootstrap block
|   |   |-- review.ts            # Review methodology
|   |   |-- design.ts            # Design methodology
|   |   |-- constants.ts         # Paths, co-author trailers
|   |   |-- types.ts             # TemplateContext interface
|   |   |-- index.ts             # Placeholder registry
|   |
|-- BROWSE ENGINE
|   |-- browse/src/              # Bun server + Playwright commands
|   |-- browse/dist/             # Compiled binary (~58MB)
|
|-- CLI UTILITIES
|   |-- bin/gstack-config        # Key-value config store
|   |-- bin/gstack-update-check  # Version check
|   |-- bin/gstack-repo-mode     # solo/collaborative detection
|   |-- bin/gstack-slug          # Project slug generator
|   |-- bin/gstack-review-log    # Review result persistence
|   |-- bin/gstack-analytics     # Usage analytics viewer
|   |-- bin/gstack-telemetry-*   # Telemetry infrastructure
|
|-- TESTING
|   |-- test/skill-validation.test.ts    # Tier 1: static (free)
|   |-- test/skill-llm-eval.test.ts      # Tier 3: LLM judge ($0.15)
|   |-- test/skill-e2e-*.test.ts         # Tier 2: full E2E ($3.85)
|   |-- test/helpers/session-runner.ts   # Spawns real Claude sessions
|   |-- test/helpers/llm-judge.ts        # Sonnet-based quality scoring
|
|-- MULTI-HOST
|   |-- .agents/skills/          # Generated Codex/Kiro/Gemini variants
|   |-- agents/openai.yaml       # Root Codex agent config
|
|-- DISTRIBUTION
|   |-- setup                    # One-command installer
|   |-- package.json             # Build scripts
```

### The Key Insight: Code-Generated Documentation

Most skill repos have a fatal flaw: **docs drift from code**. G-Stack solved this with a compilation pipeline:

```
SKILL.md.tmpl (human prose + {{PLACEHOLDERS}})
     |
     v
gen-skill-docs.ts (reads source code, resolves placeholders)
     |
     v
SKILL.md (committed, CI-validated, always accurate)
```

This means:
- If a browse command exists in code, it appears in docs
- If a command is removed from code, it disappears from docs
- CI catches stale docs before merge

---

## 3. The 7 Pillars: Why G-Stack Works

### Pillar 1: Opinionated Philosophy (ETHOS.md)

G-Stack doesn't just have skills - it has a **belief system** that gets injected into every agent:

- **"Boil the Lake"** - Always do the complete thing when AI makes the marginal cost near-zero. The last 10% of completeness costs seconds, not days. Never recommend shortcuts.
- **"Search Before Building"** - Three layers: Layer 1 (tried-and-true), Layer 2 (new-and-popular, scrutinize it), Layer 3 (first principles - prize above all).
- **"The Eureka Moment"** - When first-principles reasoning contradicts conventional wisdom, name it and celebrate it.
- **"Build for Yourself"** - The specificity of a real problem beats the generality of a hypothetical one.

These aren't just words in a README. They're **compiled into every skill's preamble** and actively shape agent behavior.

### Pillar 2: Phase-Based Workflows with Hard Gates

Every complex skill follows this pattern:

```
Phase 1: Context Gathering (read code, understand state)
Phase 2: Analysis (classify, prioritize, triage)
Phase 3: Action (make changes, run tests)
Phase 4: Verification (prove it works with evidence)
Phase 5: Report (structured output with status)
```

Critical behavioral constraints:
- **Iron Laws** - "NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST" (/investigate)
- **Hard Gates** - "Do NOT invoke any implementation skill" (/office-hours)
- **Stop Conditions** - Explicit lists of when to stop vs. continue (/ship)
- **3-Strike Rules** - "If 3 hypotheses fail, STOP" (/investigate)

### Pillar 3: Structured User Interaction (AskUserQuestion Format)

Every user question follows the same format across all skills:

1. **Re-ground** - State project, branch, current task (assume user hasn't looked in 20 min)
2. **Simplify** - Explain in plain English a 16-year-old could follow
3. **Recommend** - Always state recommendation with reason + Completeness score (X/10)
4. **Options** - Lettered with effort estimates showing both human and AI time

### Pillar 4: Completion Status Protocol

Every skill ends with one of:
- **DONE** - All steps completed with evidence
- **DONE_WITH_CONCERNS** - Completed with listed issues
- **BLOCKED** - Cannot proceed (what was tried, what to do next)
- **NEEDS_CONTEXT** - Missing information (exactly what's needed)

Plus escalation rules:
- 3 failed attempts -> STOP and escalate
- Uncertain about security -> STOP and escalate
- "Bad work is worse than no work"

### Pillar 5: Persistent State via Browse Daemon

The browse engine gives skills persistent browser state:
- First call: ~3s startup
- Every call after: ~100-200ms
- Cookies, tabs, localStorage persist between commands
- Auto-shutdown after 30 min idle
- Security: localhost-only, bearer token auth, no plaintext cookie storage

### Pillar 6: Multi-Host Portability

Same skills work across 4 AI coding agents:
- Claude Code (primary)
- OpenAI Codex
- Amazon Kiro
- Google Gemini

Achieved through the template system that generates host-specific variants with correct paths and conventions.

### Pillar 7: Skills Test Like Software

Three-tier testing:
- **Tier 1 (Free, <2s)** - Static validation: parse every `$B` command in SKILL.md, validate against registry
- **Tier 2 ($3.85, ~20min)** - E2E: spawn real Claude sessions, run each skill, check for errors
- **Tier 3 ($0.15, ~30s)** - LLM-as-judge: Sonnet scores docs on clarity/completeness/actionability

---

## 4. SKILL.md Anatomy

Every SKILL.md follows this structure:

### Frontmatter (YAML)

```yaml
---
name: skill-name
preamble-tier: 1-4          # Controls how much preamble scaffolding
version: X.Y.Z
description: |
  Multi-line description. First line is the summary.
  Second line adds trigger phrases ("Use when asked to...").
  Third line adds proactive triggers.
  Optional: "For X, use /other-skill instead."
benefits-from: [other-skill]  # Optional: skills that enhance this one
allowed-tools:                 # Explicit tool whitelist
  - Bash
  - Read
  - Edit
  - AskUserQuestion
  - WebSearch
hooks:                         # Optional: pre-tool-use hooks
  PreToolUse:
    - matcher: "Edit"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/../freeze/bin/check-freeze.sh"
---
```

### Key Frontmatter Patterns:

1. **`preamble-tier`** - The most important field. Controls complexity:
   - T1: core + upgrade + telemetry + trimmed voice (browse, benchmark)
   - T2: T1 + full voice + AskUserQuestion format + completeness (investigate, retro)
   - T3: T2 + repo-mode + search-before-building (office-hours, plan reviews)
   - T4: T3 + test-failure-triage (ship, qa, review)

2. **`description`** - Does double duty: human-readable AND used as the skill trigger. Format:
   ```
   Line 1: What it does (imperative)
   Line 2: "Use when asked to..." (trigger phrases)
   Line 3: "Proactively suggest when..." (auto-invoke conditions)
   Line 4: "For X, use /other instead" (disambiguation)
   ```

3. **`allowed-tools`** - Explicit whitelist prevents tool misuse. Office-hours gets no Write (design docs only). Browse gets no Edit (read-only testing).

4. **`hooks`** - Pre-tool checks. The freeze/guard system uses hooks to block edits outside a scoped directory.

### Body Structure

```markdown
{{PREAMBLE}}                    # Auto-injected based on tier

# Skill Title                   # One-line role statement

## Iron Law / Hard Gate          # The ONE constraint that cannot be violated

## Setup                         # Parameter parsing, binary discovery, state checks

## Phase 1-N                     # Numbered phases with clear inputs/outputs

## Output Structure              # Exact file/directory structure for outputs

## Important Rules               # Behavioral constraints as bullet points
```

---

## 5. The Template System

### Placeholders

Templates use `{{PLACEHOLDER}}` syntax resolved by `scripts/resolvers/`:

| Placeholder | Source | Purpose |
|-------------|--------|---------|
| `{{PREAMBLE}}` | preamble.ts | Universal skill setup (tiers 1-4) |
| `{{BROWSE_SETUP}}` | browse.ts | Binary discovery + setup |
| `{{BASE_BRANCH_DETECT}}` | preamble.ts | Dynamic main/master detection |
| `{{COMMAND_REFERENCE}}` | commands.ts | Auto-generated command table |
| `{{SNAPSHOT_FLAGS}}` | snapshot.ts | Flag reference with examples |
| `{{QA_METHODOLOGY}}` | testing.ts | Shared QA phases |
| `{{DESIGN_METHODOLOGY}}` | design.ts | Shared design audit methodology |
| `{{TEST_BOOTSTRAP}}` | testing.ts | Test framework detection |
| `{{TEST_FAILURE_TRIAGE}}` | preamble.ts | How to handle failing tests |
| `{{REVIEW_DASHBOARD}}` | review.ts | Review readiness status |
| `{{CO_AUTHOR_TRAILER}}` | constants.ts | Git commit co-author line |
| `{{CODEX_PLAN_REVIEW}}` | codex-helpers.ts | Cross-model review |

### Resolver Architecture

Each resolver is a pure function: `(ctx: TemplateContext) => string`

```typescript
interface TemplateContext {
  host: 'claude' | 'codex' | 'kiro';
  skillName: string;
  tmplPath: string;
  preambleTier: number;
  paths: {
    binDir: string;
    skillRoot: string;
    localSkillRoot: string;
    browseDir: string;
  };
}
```

The context adapts paths per host:
- Claude: `~/.claude/skills/gstack/`
- Codex: `$HOME/.codex/skills/gstack/` with runtime detection
- Kiro: `~/.kiro/skills/gstack/`

### Why This Works

1. **Single source of truth** - Commands defined in code, referenced in docs automatically
2. **Shared blocks** - QA methodology written once, used by /qa and /qa-only
3. **Host adaptation** - Same template generates correct paths for each AI agent
4. **CI validation** - `gen:skill-docs --dry-run` + `git diff --exit-code` catches staleness

---

## 6. The Preamble Architecture

The preamble is the most sophisticated piece. Every skill runs it first. Here's what it does:

### Phase 1: Environment Detection (Bash Block)

```bash
# 1. Update check
_UPD=$(gstack-update-check 2>/dev/null || true)

# 2. Session tracking (count active sessions)
touch ~/.gstack/sessions/"$PPID"
_SESSIONS=$(find ~/.gstack/sessions -mmin -120 -type f | wc -l)

# 3. User preferences
_CONTRIB=$(gstack-config get gstack_contributor)
_PROACTIVE=$(gstack-config get proactive)

# 4. Branch detection
_BRANCH=$(git branch --show-current)

# 5. Repo mode (solo vs collaborative)
source <(gstack-repo-mode)

# 6. Telemetry (local JSONL always, remote opt-in)
echo '{"skill":"name","ts":"..."}' >> ~/.gstack/analytics/skill-usage.jsonl
```

### Phase 2: Behavioral Directives (Generated per tier)

**Tier 1 (all skills):**
- Upgrade notification
- Lake intro (one-time "Boil the Lake" philosophy introduction)
- Telemetry opt-in prompt (one-time, 3-step: community -> anonymous -> off)
- Proactive behavior prompt (one-time)
- Voice directive (trimmed for T1, full for T2+)
- Contributor mode
- Completion status protocol

**Tier 2 adds:**
- AskUserQuestion format specification
- Completeness Principle with effort reference table

**Tier 3 adds:**
- Repo ownership mode (solo: fix everything, collaborative: flag only)
- Search Before Building with 3-layer knowledge model

**Tier 4 adds:**
- Test failure ownership triage (in-branch vs pre-existing classification)

### Why the Tier System Works

- **Lightweight skills (T1)** don't need 2000 words of behavioral scaffolding
- **Complex workflows (T4)** need sophisticated error handling and test triage
- Each tier is additive - T4 includes everything from T1-T3
- Adding a new behavioral directive to one tier automatically propagates to all skills at that tier or above

---

## 7. Prompt Engineering Patterns

### Pattern 1: Role Assignment with Constraints

```markdown
You are a QA engineer AND a bug-fix engineer.
```

Not just "you are a QA tester" - the dual role tells the agent it both finds AND fixes.

```markdown
You are a YC office hours partner.
**HARD GATE:** Do NOT invoke any implementation skill, write any code...
```

The hard gate prevents the agent from "helpfully" doing more than asked.

### Pattern 2: Tables for Decision Logic

Instead of nested if/else:
```markdown
| Parameter | Default | Override example |
|-----------|---------|-----------------|
| Target URL | (auto-detect) | `https://myapp.com` |
| Tier | Standard | `--quick`, `--exhaustive` |
```

### Pattern 3: Explicit Stop Conditions

```markdown
**Only stop for:**
- On the base branch (abort)
- Merge conflicts that can't be auto-resolved
- In-branch test failures

**Never stop for:**
- Uncommitted changes (always include them)
- Version bump choice (auto-pick MICRO)
- CHANGELOG content (auto-generate from diff)
```

This is critical - without explicit "never stop for" lists, agents pause for trivial confirmations.

### Pattern 4: Anti-Pattern Lists

```markdown
**Anti-patterns:**
- "Choose B -- it covers 90% with less code." (If A is 70 lines more, choose A.)
- "Let's defer tests to a follow-up PR." (Tests are the cheapest lake to boil.)
```

### Pattern 5: Evidence-Based Verification

```markdown
**Rationalization prevention:**
- "Should work now" -> RUN IT.
- "I'm confident" -> Confidence is not evidence.
- "I already tested earlier" -> Code changed since then. Test again.
```

### Pattern 6: Structured Output Templates

```markdown
DEBUG REPORT
================
Symptom:         [what the user observed]
Root cause:      [what was actually wrong]
Fix:             [what was changed, with file:line references]
Evidence:        [test output, reproduction attempt]
Status:          DONE | DONE_WITH_CONCERNS | BLOCKED
================
```

### Pattern 7: Cross-Skill References

```markdown
Suggest adjacent gstack skills when relevant:
- Brainstorming -> /office-hours
- Strategy -> /plan-ceo-review
- Architecture -> /plan-eng-review
- QA -> /qa
- Shipping -> /ship
```

This creates a **workflow graph** where each skill knows what comes before and after it.

### Pattern 8: Bash Block Self-Containment

```markdown
Rules:
- Each bash code block runs in a separate shell -- variables do not persist
- Use natural language for logic and state between blocks
- Keep bash blocks self-contained
- Express conditionals as English, not nested if/elif/else
```

---

## 8. The Voice System

Two tiers of voice directives:

### Tier 1 (Lightweight Skills)
```
Tone: direct, concrete, sharp, never corporate, never academic.
Sound like a builder, not a consultant. Name the file, the function, the command.
No filler, no throat-clearing.
```

### Tier 2+ (Full Voice - "Garry Tan" Voice)
A comprehensive voice specification that includes:
- Core belief system ("there is no one at the wheel")
- Tone calibration ("YC partner energy for strategy, senior eng energy for code review")
- Banned vocabulary (delve, crucial, robust, comprehensive, nuanced...)
- Banned phrases ("here's the kicker", "let me break this down"...)
- Writing rules (no em dashes, short paragraphs, name specifics)
- Humor directive ("dry observations about the absurdity of software")
- Final test ("does this sound like a real builder talking to a builder?")

### Why This Works

The voice system prevents "AI slop" - that generic, corporate, overly-enthusiastic tone that makes AI outputs feel artificial. By banning specific words and phrases, and by providing a clear persona model, the output reads like a human engineer wrote it.

---

## 9. Agent Orchestration

### Skill Chaining

Skills form a natural pipeline:

```
/office-hours (brainstorm)
     |
     v
/plan-ceo-review (strategy)
     |
     v
/plan-eng-review (architecture)
     |
     v
/plan-design-review (design)
     |
     v
[implementation]
     |
     v
/review (code review)
     |
     v
/qa (test + fix)
     |
     v
/ship (push + PR)
     |
     v
/document-release (sync docs)
     |
     v
/land-and-deploy (merge + deploy)
     |
     v
/canary (monitor)
     |
     v
/retro (retrospective)
```

### Auto-Invocation

Skills can auto-invoke each other:
- `/ship` automatically runs `/review` as Step 3.5
- `/ship` automatically runs `/document-release` as Step 8.5
- `/autoplan` chains CEO -> Design -> Eng reviews sequentially

### Cross-Model Review

The `/codex` skill provides a "second opinion" from a different AI:
- Code review mode: independent diff review
- Challenge mode: adversarial testing
- Consult mode: ask anything with session continuity

---

## 10. The Browse Daemon

### Architecture

```
Claude Code -> $B command -> CLI binary (Bun compiled)
                              |
                              v
                         HTTP POST to localhost:PORT
                              |
                              v
                         Bun.serve() server
                              |
                              v
                         Playwright CDP -> Chromium
```

### Why It's Fast

- Compiled Bun binary (~58MB, no node_modules)
- Long-lived Chromium process (no cold start per command)
- HTTP API (not MCP - lighter on tokens, debuggable with curl)
- Ref system (@e1, @e2) avoids CSS selectors entirely

### The Ref System

Instead of fragile CSS selectors:
1. Agent runs `$B snapshot -i`
2. Server walks ARIA accessibility tree
3. Assigns sequential refs: @e1, @e2, @e3...
4. Agent uses `$B click @e3` - server resolves to Playwright Locator
5. Refs are cleared on navigation (stale refs fail loudly)

---

## 11. Testing & Eval System

### Three Tiers

| Tier | Cost | Speed | What |
|------|------|-------|------|
| 1 - Static | Free | <2s | Parse $B commands, validate against registry |
| 2 - E2E | ~$3.85 | ~20min | Spawn real Claude sessions, run skills |
| 3 - LLM Judge | ~$0.15 | ~30s | Sonnet scores doc quality |

### E2E Session Runner

Tests spawn `claude -p` as a subprocess:
1. Write prompt to temp file
2. Spawn `cat prompt | claude -p --output-format stream-json --verbose`
3. Stream NDJSON for real-time progress
4. Race against timeout
5. Parse structured results

### Diff-Based Test Selection

Each test declares file dependencies in `touchfiles.ts`. Only tests affected by the current diff run. `EVALS_ALL=1` forces all tests.

### Two-Tier Classification

- **Gate** - Safety guardrails, deterministic functional tests (block merge)
- **Periodic** - Quality benchmarks, non-deterministic tests (weekly cron)

---

## 12. Distribution & Installation

### One-Command Setup

```bash
git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
cd ~/.claude/skills/gstack && ./setup
```

The `setup` script:
1. Builds browse binary via `bun build --compile`
2. Installs Playwright Chromium
3. Creates `~/.gstack/` state directory
4. Symlinks individual skills into the host's skill directory
5. Handles naming (flat `/qa` vs namespaced `/gstack-qa`)
6. Auto-detects which hosts are installed (Claude, Codex, Kiro)

### Multi-Host Support

The same repo serves 4 AI agents:
- **Claude Code**: SKILL.md files used directly
- **OpenAI Codex**: Generated variants in `.agents/skills/` with Codex paths
- **Amazon Kiro**: sed-rewritten paths from Codex variants
- **Google Gemini**: Same as Codex via `.agents/skills/`

---

## 13. Extracted Templates

### Template A: SKILL.md Template

```yaml
---
name: {{SKILL_NAME}}
preamble-tier: {{1-4}}
version: {{X.Y.Z}}
description: |
  {{One-line summary of what this skill does.}}
  {{Trigger phrases: "Use when asked to..."}}
  {{Proactive trigger: "Proactively suggest when..."}}
  {{Disambiguation: "For X, use /other instead."}}
allowed-tools:
  - Bash
  - Read
  - {{additional tools as needed}}
---

{{PREAMBLE}}

# {{Skill Title}}

## Iron Law / Hard Gate

{{THE ONE RULE THAT CANNOT BE VIOLATED}}

---

## Setup

**Parse the user's request for these parameters:**

| Parameter | Default | Override example |
|-----------|---------|-----------------|
| {{param}} | {{default}} | {{example}} |

---

## Phase 1: {{Context/Investigation}}

{{Steps to gather context and understand the problem}}

---

## Phase 2: {{Analysis/Classification}}

{{Steps to analyze, classify, and prioritize}}

---

## Phase 3: {{Action/Implementation}}

{{Steps to take action, with explicit stop conditions}}

---

## Phase 4: {{Verification}}

{{Steps to prove the work was done correctly - with evidence}}

---

## Phase 5: {{Report/Output}}

{{Structured output format}}

---

## Important Rules

- {{Rule 1 - what to never do}}
- {{Rule 2 - what to always do}}
- {{Rule 3 - when to stop}}
- {{Rule 4 - escalation conditions}}
```

### Template B: AskUserQuestion Format

```markdown
## AskUserQuestion Format

1. **Re-ground:** State project, branch, current task (1-2 sentences)
2. **Simplify:** Plain English a smart 16-year-old could follow
3. **Recommend:** `RECOMMENDATION: Choose [X] because [reason]`
   Include `Completeness: X/10` for each option
4. **Options:** `A) ... B) ... C) ...` with effort: `(human: ~X / CC: ~Y)`
```

### Template C: Completion Status Protocol

```markdown
## Completion Status

- **DONE** - All steps completed with evidence
- **DONE_WITH_CONCERNS** - Completed, concerns listed
- **BLOCKED** - Cannot proceed. What was tried, what to do next.
- **NEEDS_CONTEXT** - Missing info. Exactly what's needed.

### Escalation
- 3 failed attempts -> STOP and escalate
- Uncertain about security -> STOP and escalate
- Bad work is worse than no work
```

### Template D: Voice Directive

```markdown
## Voice

**Tone:** direct, concrete, sharp, never corporate, never academic.
Sound like a builder, not a consultant.

**Writing rules:**
- No em dashes (use commas, periods, "...")
- No AI vocabulary (delve, crucial, robust, comprehensive, nuanced...)
- No filler phrases ("here's the kicker", "let me break this down"...)
- Short paragraphs. Mix one-sentence with 2-3 sentence runs.
- Name the file, the function, the line number.
- End with what to do. Give the action.
```

### Template E: Repo Structure

```
repo/
|-- ETHOS.md              # Builder philosophy
|-- ARCHITECTURE.md       # Technical decisions and why
|-- CLAUDE.md             # Dev commands, conventions, project structure
|-- CONTRIBUTING.md       # How to contribute
|-- CHANGELOG.md          # User-facing release notes
|-- VERSION               # 4-digit semver (MAJOR.MINOR.PATCH.MICRO)
|-- TODOS.md              # Prioritized work items (P0-P4)
|
|-- {skill}/
|   |-- SKILL.md.tmpl     # Template (edit this)
|   |-- SKILL.md           # Generated (don't edit)
|
|-- scripts/
|   |-- gen-skill-docs.ts  # Template compiler
|   |-- resolvers/         # Shared template blocks
|
|-- bin/                   # CLI utilities
|-- test/                  # Validation + E2E + LLM judge
|-- setup                  # One-command installer
|-- package.json           # Build scripts
```

### Template F: Effort Compression Table

```markdown
| Task type | Human team | AI-assisted | Compression |
|-----------|-----------|-------------|-------------|
| Boilerplate | 2 days | 15 min | ~100x |
| Tests | 1 day | 15 min | ~50x |
| Feature | 1 week | 30 min | ~30x |
| Bug fix | 4 hours | 15 min | ~20x |
| Architecture | 2 days | 4 hours | ~5x |
| Research | 1 day | 3 hours | ~3x |
```

---

## 14. Applying This to Your Own Repos

### Step 1: Adopt the File Structure

Every repo should have:
- `ETHOS.md` - Your builder philosophy (what you believe about how software should be built)
- `CLAUDE.md` - Dev commands, project structure, conventions
- `ARCHITECTURE.md` - Technical decisions and WHY
- `VERSION` - 4-digit semver
- `CHANGELOG.md` - User-facing release notes
- `TODOS.md` - Prioritized work items

### Step 2: Build a Template System

If you have more than 2 skills:
1. Create `.tmpl` files with `{{PLACEHOLDER}}` syntax
2. Write a `gen-skill-docs.ts` that resolves placeholders from source code
3. Add CI validation: `gen:skill-docs --dry-run && git diff --exit-code`

### Step 3: Implement the Preamble Pattern

Every skill should start with:
1. Session/environment detection
2. User preference loading
3. Voice directive
4. AskUserQuestion format
5. Completion status protocol

### Step 4: Use Phase-Based Workflows

Every non-trivial skill should follow:
1. Context Gathering -> 2. Analysis -> 3. Action -> 4. Verification -> 5. Report

With:
- Explicit stop conditions
- Escalation paths (3-strike rule)
- Evidence-based verification ("confidence is not evidence")
- Structured output format

### Step 5: Write a Voice Directive

Define:
- Tone (direct, concrete, never corporate)
- Banned vocabulary (AI slop words)
- Banned phrases
- Writing rules
- Final test ("does this sound like X?")

### Step 6: Test Skills Like Software

Minimum:
- Tier 1: Static validation (parse commands, check references)
- Tier 3: LLM-as-judge (does the output meet quality standards?)

Ideal:
- Tier 2: E2E (spawn real agent sessions, verify end-to-end)

### Step 7: Ship Distribution

- One-command `setup` script
- Symlink-based skill registration
- Compiled binaries where possible (no node_modules at runtime)
- Auto-update checking

---

## Key Takeaways

1. **Skills are software, not prose.** Generate them from code, test them with CI, version them.

2. **Every skill needs structure.** Frontmatter + preamble + phases + verification + report.

3. **Opinionated > flexible.** "Boil the Lake" works because it's a clear decision framework, not a suggestion.

4. **Voice prevents AI slop.** Ban specific words. Define what the output should sound like.

5. **Persistent state changes everything.** The browse daemon turns 3s browser launches into 100ms commands.

6. **Tiered complexity.** Not every skill needs 2000 words of behavioral scaffolding. T1 for simple, T4 for complex.

7. **Explicit stop conditions.** Tell the agent exactly when to stop and when to keep going. Both are critical.

8. **Evidence over confidence.** Every verification step must produce evidence. "Should work" is not evidence.

9. **Cross-skill awareness.** Each skill knows what comes before and after it in the workflow.

10. **Test the whole system.** Unit tests catch code bugs. E2E evals catch prompt bugs. LLM judges catch quality drift.
