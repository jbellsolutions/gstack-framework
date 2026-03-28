---
name: gf-find-agents
preamble-tier: 2
version: 1.0.0
description: |
  Identify opportunities for multi-agent patterns, cross-skill chaining, and parallel agent workflows.
  Proposes agent configurations that amplify existing skills.
  Use when asked to "find agents", "what agents does this need", or "how should skills chain together".
  Proactively suggest after /gf-find-skills when the project has 5+ skills that could be orchestrated.
  For single-skill creation, use /gf-create-skills instead.
allowed-tools:
  - Bash
  - Read
  - Glob
  - Grep
  - AskUserQuestion
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

# Count existing skills
_SKILL_COUNT=$(find "$_ROOT" -name "SKILL.md" 2>/dev/null | wc -l | tr -d ' ')
echo "EXISTING SKILLS: $_SKILL_COUNT"
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

# Find Agents

Identify where multi-agent orchestration, skill chaining, and parallel execution would amplify existing skills. Propose agent configurations with pipeline diagrams and invocation triggers.

## Iron Law

**NO AGENT PROPOSALS WITHOUT AT LEAST 2 EXISTING SKILLS TO CHAIN.**

Agents orchestrate skills. If the skills don't exist yet, run /gf-find-skills first.

---

## Phase 1: Load Context

Read the analysis report and existing skills before proposing any agents.

1. Find and read the analysis report:
   ```bash
   find "$_ROOT" -name "gf-analysis-report.*" -o -name "analysis-report.*" | head -5
   ```
2. Find all existing skills:
   ```bash
   find "$_ROOT" -name "SKILL.md" -exec echo "---" \; -exec head -10 {} \; -exec echo {} \;
   ```
3. Read each SKILL.md frontmatter to build a skill inventory:
   - Skill name
   - What it does (from description)
   - What tools it uses
   - What inputs it needs
   - What outputs it produces

4. If fewer than 2 skills exist, stop:
   > "This project has [N] skills. Agents need at least 2 skills to chain.
   > Run /gf-find-skills first to identify and create skills, then come back."

**Output:** Skill inventory table with name, purpose, inputs, and outputs.

---

## Phase 2: Map Skill Dependencies

Determine which skills naturally feed into other skills.

For each pair of skills, ask:
- Does skill A's output become skill B's input?
- Must skill A finish before skill B can start?
- Do they share the same files or state?

Build a dependency matrix:

```
              analyze  test  lint  review  ship  deploy
analyze         -       no    no    yes     no    no
test            no       -    no    yes     yes   no
lint            no      no     -    yes     yes   no
review          no      no    no     -      yes   no
ship            no      no    no    no       -    yes
deploy          no      no    no    no      no     -
```

Mark dependencies as:
- **HARD** -- B literally cannot run without A's output
- **SOFT** -- B works better with A's output but can run independently
- **NONE** -- No relationship

**Output:** Dependency matrix and a list of natural pipelines (chains of 3+ skills).

---

## Phase 3: Identify Orchestration Opportunities

Where would a meta-skill chain multiple skills together into a single command?

Look for these patterns:

### Pattern 1: Sequential Pipeline
Skills that always run in the same order.

```
Example from G-Stack:
  /autoplan = /plan-ceo-review -> /plan-design-review -> /plan-eng-review
  /ship = test -> review -> version-bump -> changelog -> push -> PR
```

**Evidence:** Check git history for skills that are consistently invoked together:
```bash
git log --oneline -50 | head -50
```

### Pattern 2: Gate Pattern
One skill acts as a quality gate before another can proceed.

```
Example from G-Stack:
  /land-and-deploy = merge PR -> wait for CI -> verify production health
  /review = lint check -> security check -> code review -> approval gate
```

### Pattern 3: Fan-out/Fan-in
One trigger spawns multiple skills, results merge back.

```
Example:
  /preflight = [test, lint, type-check, security-scan] -> merge results -> go/no-go
```

### Pattern 4: Iterative Loop
A skill runs, checks results, and re-runs until a condition is met.

```
Example from G-Stack:
  /qa = test -> fix -> re-test -> fix -> re-test (until health score passes)
  /skill-optimize = run skill -> evaluate -> mutate -> re-run (until assertions pass)
```

For each pattern found, note which skills participate and what the trigger would be.

---

## Phase 4: Identify Parallel Opportunities

What tasks could run simultaneously to save time?

Check each skill pair for independence:
- Do they read/write different files?
- Do they have no data dependencies?
- Could they run in separate processes?

Common parallel groups:
- **Static analysis:** lint + type-check + format-check
- **Testing:** unit tests + integration tests (if independent)
- **Validation:** schema validation + data validation + config validation
- **Documentation:** API docs + README update + changelog

For each parallel group:
- List the skills
- Estimate time savings (sequential vs parallel)
- Note any shared state that would prevent parallelism

**Output:** List of parallel groups with time savings estimates.

---

## Phase 5: Identify Cross-Model Opportunities

Where would a second AI opinion add value?

Look for these scenarios:

### Independent Review
Like G-Stack's /codex, where a different model reviews the same work.

```
Example:
  Developer writes code -> Claude reviews -> Codex independently reviews
  Both must pass for merge.
```

Best candidates:
- Security-sensitive code (auth, crypto, permissions)
- Architecture decisions (get a second perspective)
- User-facing content (catch tone/clarity issues)

### Adversarial Testing
One agent tries to break what another built.

```
Example from G-Stack:
  /codex challenge = Codex tries to find bugs in your code
```

### Consensus Decisions
Multiple models vote on ambiguous choices.

```
Example:
  "Should we use approach A or B?"
  -> Claude says A (with reasoning)
  -> Codex says B (with reasoning)
  -> Present both to user with evidence
```

For each opportunity, note: which skills benefit, what the second model adds, and whether the cost is justified.

---

## Phase 6: Propose Agent Configurations

For each opportunity found in Phases 3-5, output this format:

```
### [agent-name]

**Type:** orchestrator | parallel-runner | gate | loop | cross-model
**Skills chained:** skill-a -> skill-b -> skill-c
**When to invoke:** "trigger phrase 1", "trigger phrase 2"
**Priority:** P0-P4

**Pipeline:**

  +----------+     +----------+     +----------+
  | skill-a  | --> | skill-b  | --> | skill-c  |
  +----------+     +----------+     +----------+
                        |
                   [gate: pass?]
                     /       \
                   yes        no
                    |          |
               +--------+  +-------+
               | ship   |  | fix   |
               +--------+  +-------+

**What it does:**
2-3 sentences explaining the agent's behavior and value.

**Estimated savings:**
Without agent: X minutes (manual invocation of each skill)
With agent: Y minutes (single command)
```

Sort by priority. Group by type (orchestration, parallel, cross-model).

---

## Phase 7: User Approval

Present the top agent proposals via AskUserQuestion.

Format:

1. **Re-ground:** "Analyzing [project] on [branch]. Found [N] agent opportunities across [M] existing skills."
2. **Simplify:** "Here are the highest-value agent configurations. These chain your existing skills into single commands."
3. **Recommend:** RECOMMENDATION: Start with [top 2-3] because [reasons]. Completeness: X/10
4. **Options:**
   - A) Create top 2 agents (highest impact) ... effort: AI ~30 min
   - B) Create top 4 agents (orchestration + parallel) ... effort: AI ~1 hour
   - C) Create all proposed agents ... effort: AI ~2 hours
   - D) Let me pick specific ones
   - E) Skip agent creation, just save the analysis

After approval, create the agent skill files or hand off to /gf-create-skills.

---

## Phase 8: Verification

**IRON LAW: NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE.**

1. Verify every proposed agent chains at least 2 real, existing skills.
2. Verify no proposed agent duplicates an existing orchestration skill.
3. Verify pipeline diagrams match the actual skill dependencies from Phase 2.
4. Verify parallel groups have no hidden data dependencies.

**Rationalization prevention:**
- "This agent would be useful" -> Show which skills it chains and prove they exist
- "Parallelism always helps" -> Check for shared state that prevents it
- "A second model always adds value" -> Justify the cost with a concrete scenario

---

## Phase 9: Report

```
REPORT
================================
Task:            Find agents for [project]
Skills analyzed: [N] existing skills
Dependencies:    [N] hard, [M] soft
Agents proposed: [total]
  Orchestrators: [n]
  Parallel:      [n]
  Gates:         [n]
  Loops:         [n]
  Cross-model:   [n]
Top 3:           [agent1], [agent2], [agent3]
User decision:   [what they chose]
Status:          DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
================================
```

---

## Completion Status

- **DONE** -- All opportunities analyzed, agents proposed, user approved selections
- **DONE_WITH_CONCERNS** -- Completed but some skill dependencies unclear
- **BLOCKED** -- Fewer than 2 skills exist. Run /gf-find-skills first.
- **NEEDS_CONTEXT** -- Missing analysis report or skill inventory incomplete

### Escalation
- Fewer than 2 skills -> STOP, redirect to /gf-find-skills
- 3 failed attempts to map dependencies -> STOP and escalate
- Uncertain about parallel safety -> Flag as DONE_WITH_CONCERNS
- Bad agents are worse than no agents

---

## Important Rules

- Never propose agents without at least 2 existing skills to chain
- Never propose more than 10 agents (focus beats breadth)
- Always include ASCII pipeline diagrams for every agent
- Always verify parallel groups have no shared state conflicts
- Always present time savings estimates (without agent vs with agent)
- 3-strike rule: if 3 proposed agents have no real skill chains, stop and re-analyze
