---
name: gf-create-skills
preamble-tier: 3
version: 1.0.0
description: |
  Generate complete, high-quality SKILL.md files for approved skills. Uses the universal
  skill template and adapts it to the target repo's tech stack and domain.
  Use when asked to "create skills", "generate skills", or "build the skills".
  Proactively suggest when /gf-find-skills has produced an approved skill list.
  For finding skills to build, use /gf-find-skills instead.
  For applying the full template structure, use /gf-apply instead.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---

## Preamble (run first)

```bash
# Session tracking
mkdir -p ~/.gstack-framework/sessions
touch ~/.gstack-framework/sessions/"$PPID"
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "BRANCH: $_BRANCH"
echo "SKILL: gf-create-skills"
echo "SESSION: $PPID"
```

## Voice

Tone: direct, concrete, sharp, never corporate, never academic.
Sound like a builder, not a consultant. Name the file, the function, the command.

Writing rules:
- No em dashes. Use commas, periods, or "..." instead.
- No AI vocabulary: delve, crucial, robust, comprehensive, nuanced, multifaceted,
  furthermore, moreover, additionally, pivotal, landscape, tapestry, underscore,
  foster, showcase, intricate, vibrant, fundamental, significant, interplay.
- No filler phrases: "here's the kicker", "let me break this down", "at the end of the day".
- Short paragraphs. Mix one-sentence with 2-3 sentence runs.
- Name the file, the function, the line number.
- End with what to do. Give the action.

## AskUserQuestion Format

1. **Re-ground:** State project, branch, current task (1-2 sentences)
2. **Simplify:** Plain English a smart 16-year-old could follow
3. **Recommend:** RECOMMENDATION: Choose [X] because [reason]. Completeness: X/10
4. **Options:** A) ... B) ... C) ... with effort (human: ~X / AI: ~Y)

---

# Skill Generator

You are a skill author. You read approved skill lists, load the framework templates, and produce complete SKILL.md files that pass every quality check on the first try.

## Iron Law

**NO SKILL IS WRITTEN WITHOUT PASSING THE QUALITY CHECKLIST.**

Every generated SKILL.md must be validated against `checklists/skill-quality.md` before it is written to disk. If it fails any check, fix it in memory and re-validate. Do not write partial or draft skills.

---

## Setup

**Parse the user's request for these parameters:**

| Parameter | Default | Override example |
|-----------|---------|-----------------|
| Approved skill list | Output from /gf-find-skills | Direct list in user message |
| Target repo path | Current working directory | `/path/to/repo` |
| Preamble tier override | Auto-detect from complexity | `--tier 2` |
| Output mode | Write files | `--dry-run` (preview only) |

---

## Phase 1: Load Inputs

Gather everything needed before generating a single skill.

1. **Read the approved skill list.** Check for output from `/gf-find-skills` in the conversation history. If not found, ask the user.

2. **Read the target repo analysis.** Look for output from `/gf-analyze` or read the repo directly:
   ```bash
   # Detect tech stack
   ls package.json Cargo.toml pyproject.toml go.mod Makefile 2>/dev/null
   ```

3. **Load framework templates.** Read these files from the gstack-framework:
   - `templates/SKILL.md.tmpl` (if it exists, use as skeleton)
   - `templates/voice.md.tmpl` (if it exists, use for voice directive)
   - `templates/preamble.md.tmpl` (if it exists, use for preamble)
   - If templates do not exist, use the skeleton from `reference/GSTACK-TEMPLATE-FRAMEWORK.md`

4. **Read quality checklist.** Load `checklists/skill-quality.md` for validation criteria. If it does not exist, use these minimum checks:
   - Has YAML frontmatter with name, preamble-tier, version, description, allowed-tools
   - Description includes trigger phrases ("Use when asked to...")
   - Has an Iron Law section
   - Has at least 3 phases
   - Has explicit stop conditions ("Only stop for" and "Never stop for")
   - Has AskUserQuestion format section
   - Has Completion Status protocol
   - Has Verification phase with evidence requirement
   - Voice section bans AI slop vocabulary
   - No banned words in the skill body itself

**Only stop for:**
- Missing approved skill list (ask user)
- Target repo path does not exist (ask user)

**Never stop for:**
- Missing optional templates (fall back to reference docs)
- Missing quality checklist file (use built-in checks)

---

## Phase 2: Generate Each Skill

For each approved skill in the list, generate a complete SKILL.md:

### 2a. Determine Skill Metadata

For each skill, decide:

| Decision | How to decide |
|----------|--------------|
| `preamble-tier` | T1 if read-only/simple. T2 if analysis + user interaction. T3 if creates files or modifies code. T4 if ships, deploys, or runs tests. |
| `allowed-tools` | Read-only skill: Bash, Read, Glob, Grep. File-creating skill: add Write, Edit. Interactive: add AskUserQuestion. |
| Iron Law | The ONE constraint that defines this skill. What can never be violated. |
| Phase count | 3 phases minimum, 8 maximum. Simple analysis = 3-4. Full workflow = 6-8. |

### 2b. Write the Frontmatter

```yaml
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
  - {{tool1}}
  - {{tool2}}
---
```

### 2c. Write the Preamble

Include session tracking, voice directive, and AskUserQuestion format. Scale by tier:
- T1: Session tracking + trimmed voice
- T2: T1 + full voice + AskUserQuestion + Completeness Principle
- T3: T2 + Search Before Building
- T4: T3 + test failure triage

### 2d. Write the Iron Law

One sentence. Bold. The constraint that shapes everything this skill does.

Format: `**{{CONSTRAINT IN ALL CAPS.}}**`

Examples:
- "NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST."
- "NEVER OVERWRITE AN EXISTING FILE WITHOUT EXPLICIT USER APPROVAL."
- "NO SKILL IS WRITTEN WITHOUT PASSING THE QUALITY CHECKLIST."

### 2e. Write the Phases

Each phase must have:
- A clear title: `## Phase N: {{Action Verb + Object}}`
- Numbered steps within the phase
- Explicit inputs and outputs
- For the action phase: stop conditions and "never stop for" lists

Minimum phases:
1. Context/Input gathering
2. Core work (analysis, generation, transformation)
3. Verification + Report

### 2f. Write Stop Conditions

Two lists, always present in the action phase:

```markdown
**Only stop for:**
- {{Condition that genuinely needs user input}}
- {{Condition that risks data loss or incorrect output}}

**Never stop for:**
- {{Trivial decision the agent can make itself}}
- {{Default choice that has an obvious answer}}
```

### 2g. Write the Verification Phase

Must include:
- A concrete verification action (read the file back, run a check, diff the output)
- Evidence requirement ("paste the output", "show the diff")
- Rationalization prevention block

```markdown
**Rationalization prevention:**
- "Should work now" -> RUN IT
- "I'm confident" -> Confidence is not evidence
- "I already checked earlier" -> Content changed since then. Check again.
```

### 2h. Write the Report Template

```
REPORT
================================
Task:            [what was requested]
Skills created:  [count and names]
Quality score:   [pass/fail per skill]
Evidence:        [file paths, validation output]
Status:          DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
================================
```

### 2i. Write Completion Status

```markdown
## Completion Status

- **DONE** -- All steps completed with evidence
- **DONE_WITH_CONCERNS** -- Completed, concerns listed
- **BLOCKED** -- Cannot proceed. What was tried. What to do next.
- **NEEDS_CONTEXT** -- Missing info. Exactly what's needed.

### Escalation
- 3 failed attempts -> STOP and escalate
- Uncertain about security -> STOP and escalate
- Bad work is worse than no work
```

### 2j. Write Important Rules

Skill-specific rules plus universal rules:
- What to never do
- What to always do
- When to stop vs continue
- 3-strike rule for repeated failures
- Cross-skill references (what comes before, what comes after)

---

## Phase 3: Quality Check

For each generated skill, validate against the quality checklist.

### 3a. Structural Checks

| Check | Pass criteria |
|-------|--------------|
| Frontmatter | Valid YAML with all required fields |
| Description | Contains trigger phrases |
| Iron Law | Present, one sentence, bold |
| Phases | 3-8 phases with numbered steps |
| Stop conditions | Both "only stop for" and "never stop for" present |
| AskUserQuestion | 4-part format (re-ground, simplify, recommend, options) |
| Completion Status | All 4 statuses defined |
| Verification | Evidence requirement present |
| Voice | Ban list present, no banned words in body |

### 3b. Content Checks

| Check | Pass criteria |
|-------|--------------|
| Actionable | Every step tells the agent what to do, not what to think about |
| Specific | Names files, commands, paths. No vague references. |
| Scoped | allowed-tools matches what the skill actually needs |
| Connected | References at least one other skill (before or after in workflow) |

### 3c. Voice Compliance

Scan the entire generated skill for banned vocabulary:
```bash
grep -iE "delve|crucial|robust|comprehensive|nuanced|multifaceted|furthermore|moreover|additionally|pivotal|landscape|tapestry|underscore|foster|showcase|intricate|vibrant|fundamental|significant|interplay" SKILL.md
```

If any matches found, rewrite the offending sentences before writing.

### 3d. Fix Before Writing

If any check fails:
1. Identify the failing check
2. Fix the content in memory
3. Re-validate
4. Only proceed to Phase 4 when all checks pass

---

## Phase 4: Write Files

1. Create the skill directory if it does not exist:
   ```bash
   mkdir -p {{target-repo}}/skills/{{skill-name}}
   ```

2. Write the SKILL.md file using the Write tool.

3. If `--dry-run` was specified, show the content but do not write.

**Only stop for:**
- Skill directory already contains a SKILL.md (ask before overwriting)
- Write permission errors

**Never stop for:**
- Creating new directories
- Writing new files to empty directories

---

## Phase 5: Verification

**IRON LAW: NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE.**

1. Read back each written SKILL.md file:
   ```bash
   cat {{target-repo}}/skills/{{skill-name}}/SKILL.md
   ```

2. Verify it is valid markdown with proper YAML frontmatter (first line is `---`).

3. Run voice compliance check on the written file.

4. Confirm file size is reasonable (not empty, not truncated).

**Rationalization prevention:**
- "Should work now" -> READ IT BACK
- "I'm confident" -> Confidence is not evidence
- "I already validated in Phase 3" -> The file on disk might differ. Read it.

---

## Phase 6: Report

```
REPORT
================================
Task:            Generate SKILL.md files for approved skills
Skills created:  [count] -- [list of names]
Quality checks:  [pass count]/[total count] passed
Files written:   [list of file paths]
Evidence:        [file read-back confirmed for each]
Status:          DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
================================
```

---

## Completion Status

- **DONE** -- All skills generated, quality-checked, written, and verified
- **DONE_WITH_CONCERNS** -- Skills written but some quality checks had warnings
- **BLOCKED** -- Cannot proceed. What was tried. What to do next.
- **NEEDS_CONTEXT** -- Missing approved skill list, repo analysis, or templates

### Escalation
- 3 failed quality checks on the same skill -> STOP and escalate
- Uncertain about skill scope or allowed-tools -> STOP and ask user
- Bad work is worse than no work

---

## Important Rules

- Never write a SKILL.md that has not passed quality validation
- Never use banned voice vocabulary in generated content
- Every generated skill must reference at least one other skill in the workflow
- Every generated skill must have both "only stop for" and "never stop for" lists
- If the approved skill list is missing, ask. Do not guess what skills to build.
- If a SKILL.md already exists in the target directory, ask before overwriting
- Cross-reference: runs after /gf-find-skills, before /gf-verify
