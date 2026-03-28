---
name: gf-verify
preamble-tier: 2
version: 1.0.0
description: |
  Verify that all framework components were applied correctly. Checks file existence,
  skill quality, template compliance, and produces a final quality score.
  Use when asked to "verify", "check the work", or "is everything applied correctly".
  Proactively suggest after /gf-apply or /gf-create-skills has completed.
  For applying the template, use /gf-apply instead.
  For generating skills, use /gf-create-skills instead.
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
mkdir -p ~/.gstack-framework/sessions
touch ~/.gstack-framework/sessions/"$PPID"
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "BRANCH: $_BRANCH"
echo "SKILL: gf-verify"
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

# Framework Verifier

You are a quality inspector. You check that the G-Stack framework was applied correctly, score the result, and report exactly what is right, what is wrong, and what to fix.

## Iron Law

**NO VERIFICATION REPORT WITHOUT READING EVERY FILE BEING SCORED.**

Do not score a file you have not read. Do not assume a file is correct because it exists. Read it, check it, score it.

---

## Setup

**Parse the user's request for these parameters:**

| Parameter | Default | Override example |
|-----------|---------|-----------------|
| Target repo path | Current working directory | `/path/to/repo` |
| Check scope | Full (all checks) | `--only skills`, `--only structure` |
| Verbose output | false | `--verbose` (show per-check details) |

---

## Phase 1: File Existence Check

Verify all expected framework files exist.

### 1a. Required Files

Check for each file and record status:

```bash
echo "=== FILE EXISTENCE CHECK ==="
for f in ETHOS.md CLAUDE.md ARCHITECTURE.md VERSION; do
  if [ -f "$f" ]; then echo "PASS: $f exists"; else echo "FAIL: $f missing"; fi
done
```

### 1b. Recommended Files

```bash
echo "=== RECOMMENDED FILES ==="
for f in CHANGELOG.md TODOS.md CONTRIBUTING.md; do
  if [ -f "$f" ]; then echo "PASS: $f exists"; else echo "WARN: $f missing (recommended)"; fi
done
```

### 1c. Skill Directories

```bash
echo "=== SKILL FILES ==="
if [ -d "skills" ]; then
  find skills -name "SKILL.md" -type f
else
  echo "WARN: No skills/ directory found"
fi
```

### Scoring

| Check | Points | Weight |
|-------|--------|--------|
| ETHOS.md exists | 10 | Required |
| CLAUDE.md exists | 10 | Required |
| ARCHITECTURE.md exists | 10 | Required |
| VERSION exists | 5 | Required |
| CHANGELOG.md exists | 5 | Recommended |
| TODOS.md exists | 5 | Recommended |
| At least 1 SKILL.md | 5 | Recommended |

Maximum for Phase 1: 50 points.

---

## Phase 2: Skill Quality Scan

For each SKILL.md found in the repo, validate against quality criteria.

### 2a. Structural Checks (per skill)

Read each SKILL.md and verify:

| Check | Pass criteria | Points |
|-------|--------------|--------|
| YAML frontmatter | First line is `---`, valid YAML block | 2 |
| `name` field | Present and non-empty | 1 |
| `preamble-tier` field | Present, value 1-4 | 1 |
| `version` field | Present, semver format | 1 |
| `description` field | Present, contains trigger phrases | 2 |
| `allowed-tools` field | Present, is a list | 1 |
| Iron Law section | Present with bold constraint | 2 |
| Phase sections | At least 3 phases with `## Phase` headers | 3 |
| Stop conditions | "Only stop for" AND "Never stop for" present | 2 |
| AskUserQuestion format | 4-part format present | 2 |
| Completion Status | All 4 statuses defined (DONE, DONE_WITH_CONCERNS, BLOCKED, NEEDS_CONTEXT) | 2 |
| Verification phase | Evidence requirement present | 2 |
| Voice section | Present with ban list | 1 |
| Report template | Structured output format present | 1 |
| Escalation rules | 3-strike rule or equivalent | 1 |

Maximum per skill: 24 points.

### 2b. Validate Each Check

For each SKILL.md:

```bash
FILE="skills/{{skill-name}}/SKILL.md"
echo "=== CHECKING: $FILE ==="

# Frontmatter check
head -1 "$FILE" | grep -q '^---$' && echo "PASS: frontmatter delimiter" || echo "FAIL: no frontmatter"

# Required fields
grep -q '^name:' "$FILE" && echo "PASS: name field" || echo "FAIL: name field missing"
grep -q '^preamble-tier:' "$FILE" && echo "PASS: preamble-tier" || echo "FAIL: preamble-tier missing"
grep -q '^version:' "$FILE" && echo "PASS: version" || echo "FAIL: version missing"
grep -q '^description:' "$FILE" && echo "PASS: description" || echo "FAIL: description missing"
grep -q 'Use when asked to' "$FILE" && echo "PASS: trigger phrases" || echo "FAIL: no trigger phrases"
grep -q '^allowed-tools:' "$FILE" && echo "PASS: allowed-tools" || echo "FAIL: allowed-tools missing"

# Sections
grep -q '## Iron Law' "$FILE" && echo "PASS: Iron Law" || echo "FAIL: Iron Law missing"
grep -c '## Phase' "$FILE" | xargs -I{} echo "INFO: {} phases found"
grep -q 'Only stop for' "$FILE" && echo "PASS: stop conditions" || echo "FAIL: stop conditions missing"
grep -q 'Never stop for' "$FILE" && echo "PASS: never-stop list" || echo "FAIL: never-stop list missing"
grep -q 'AskUserQuestion' "$FILE" && echo "PASS: AskUserQuestion" || echo "FAIL: AskUserQuestion missing"
grep -q 'DONE_WITH_CONCERNS' "$FILE" && echo "PASS: completion status" || echo "FAIL: completion status incomplete"
grep -q 'Rationalization prevention' "$FILE" && echo "PASS: evidence requirement" || echo "FAIL: evidence requirement missing"
```

---

## Phase 3: Template Compliance

Check that generated framework files follow the templates.

### 3a. ETHOS.md Compliance

Read ETHOS.md and verify:
- Has "Completeness Principle" or "Boil the Lake" section
- Has "Search Before Building" or equivalent
- Has "Build for Yourself" or equivalent
- Contains the project name (not template placeholders like `{{PROJECT_NAME}}`)
- No unresolved `{{...}}` placeholders

### 3b. CLAUDE.md Compliance

Read CLAUDE.md and verify:
- Has "Commands" section with actual commands (not `{{PACKAGE_MANAGER}}` placeholders)
- Has "Project Structure" section with actual directory tree
- Has "Key Conventions" section
- Has "AI Effort Compression" table (recommended, not required)
- No unresolved `{{...}}` placeholders

### 3c. ARCHITECTURE.md Compliance

Read ARCHITECTURE.md and verify:
- Has "The Core Idea" or equivalent section
- Has technology choice rationale
- Has "What's Intentionally Not Here" section
- No unresolved `{{...}}` placeholders

### 3d. VERSION Compliance

```bash
# Check 4-digit semver format
cat VERSION | grep -qE '^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$' && echo "PASS: 4-digit semver" || echo "FAIL: VERSION not 4-digit semver"
```

### 3e. Placeholder Scan

```bash
echo "=== UNRESOLVED PLACEHOLDERS ==="
grep -rn '{{.*}}' ETHOS.md CLAUDE.md ARCHITECTURE.md 2>/dev/null || echo "PASS: no unresolved placeholders"
```

Scoring for Phase 3: 5 points per file checked (20 total for 4 files).

---

## Phase 4: Cross-Reference Check

Verify skills reference each other correctly.

1. **Skill reference scan:** For each SKILL.md, extract references to other skills (patterns: `/gf-`, `use /`, `run /`).

2. **Validate references:** For each referenced skill, verify the target SKILL.md exists:
   ```bash
   # Extract skill references from a SKILL.md
   grep -oE '/gf-[a-z-]+' skills/*/SKILL.md | sort -u
   ```

3. **Check for orphan skills:** Skills that are not referenced by any other skill.

4. **Check for broken references:** Skills that reference a non-existent skill.

Scoring: 5 points if all references resolve. 0 if any broken references.

---

## Phase 5: Voice Compliance

Scan all SKILL.md files for banned vocabulary.

### 5a. Banned Word Scan

```bash
echo "=== VOICE COMPLIANCE ==="
BANNED="delve|crucial|robust|comprehensive|nuanced|multifaceted|furthermore|moreover|additionally|pivotal|landscape|tapestry|underscore|foster|showcase|intricate|vibrant|fundamental|significant|interplay"

find . -name "SKILL.md" -type f | while read f; do
  HITS=$(grep -ciE "$BANNED" "$f" 2>/dev/null || echo 0)
  if [ "$HITS" -gt 0 ]; then
    echo "FAIL: $f has $HITS banned word(s)"
    grep -niE "$BANNED" "$f"
  else
    echo "PASS: $f clean"
  fi
done
```

### 5b. Banned Phrase Scan

Also check for:
- Em dashes (` -- ` is OK for status definitions, but ` --- ` as prose punctuation is not the same as markdown HR)
- "Here's the kicker"
- "Let me break this down"
- "At the end of the day"

### 5c. Framework Files Voice Check

Scan ETHOS.md, CLAUDE.md, and ARCHITECTURE.md for the same banned vocabulary.

Scoring: 10 points if all files clean. Deduct 1 point per banned word occurrence (minimum 0).

---

## Phase 6: Final Score

Calculate the overall quality score.

### Scoring Breakdown

| Category | Max Points | Weight |
|----------|-----------|--------|
| Phase 1: File Existence | 50 | 30% |
| Phase 2: Skill Quality | 24 per skill (averaged) | 25% |
| Phase 3: Template Compliance | 20 | 20% |
| Phase 4: Cross-References | 5 | 10% |
| Phase 5: Voice Compliance | 10 | 15% |

### Grade Scale

| Score | Grade | Meaning |
|-------|-------|---------|
| 90-100 | A | Ship it. Framework applied correctly. |
| 80-89 | B | Good. Minor issues to address. |
| 70-79 | C | Acceptable. Several issues need fixing. |
| 60-69 | D | Below standard. Multiple gaps need fixing. |
| 0-59 | F | Needs rework. Run /gf-apply again. |

### Calculation

```
file_score = (points_earned / 50) * 30
skill_score = (avg_skill_points / 24) * 25
template_score = (template_points / 20) * 20
crossref_score = (crossref_points / 5) * 10
voice_score = (voice_points / 10) * 15

total = file_score + skill_score + template_score + crossref_score + voice_score
```

---

## Phase 7: Report

```
VERIFICATION REPORT
================================
Project:         {{project name}}
Branch:          {{branch}}
Date:            {{date}}

FILE EXISTENCE                    [XX/50]
  ETHOS.md:        [PASS | FAIL]
  CLAUDE.md:       [PASS | FAIL]
  ARCHITECTURE.md: [PASS | FAIL]
  VERSION:         [PASS | FAIL]
  CHANGELOG.md:    [PASS | WARN]
  TODOS.md:        [PASS | WARN]
  Skills found:    [count]

SKILL QUALITY (avg)               [XX/24]
  [Per-skill breakdown if --verbose]

TEMPLATE COMPLIANCE               [XX/20]
  ETHOS.md:        [PASS | FAIL] [details]
  CLAUDE.md:       [PASS | FAIL] [details]
  ARCHITECTURE.md: [PASS | FAIL] [details]
  VERSION:         [PASS | FAIL] [details]
  Unresolved placeholders: [count]

CROSS-REFERENCES                  [XX/5]
  Valid references:   [count]
  Broken references:  [count] [list]
  Orphan skills:      [count] [list]

VOICE COMPLIANCE                  [XX/10]
  Banned words found: [count]
  [List of violations if any]

================================
FINAL SCORE:     [XX/100]
GRADE:           [A | B | C | D | F]
================================

ISSUES TO FIX:
1. [Issue description] -> [How to fix]
2. [Issue description] -> [How to fix]

NEXT STEPS:
- [What to do based on the grade]

Status: DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
================================
```

---

## Completion Status

- **DONE** -- All checks completed, score calculated, report generated
- **DONE_WITH_CONCERNS** -- Checks completed but score below 80 (Grade B threshold)
- **BLOCKED** -- Cannot read required files. What was tried. What to do next.
- **NEEDS_CONTEXT** -- Missing target repo path or scope parameters

### Escalation
- Cannot read a file that should exist -> note as FAIL, do not stop
- Score below 60 -> recommend running /gf-apply again
- Bad work is worse than no work

---

## Important Rules

- Read every file before scoring it. Never assume content from filename alone.
- Never modify any file during verification. This is a read-only skill.
- Report exact line numbers and file paths for every issue found.
- Always calculate the score. Never skip the math.
- If a SKILL.md references checklists/skill-quality.md, use it. If not, use the built-in checks in Phase 2.
- Grade honestly. A failing score is useful information. An inflated score helps nobody.
- Cross-reference: runs after /gf-apply and /gf-create-skills. Suggests /gf-apply for fixes.
- If the score is below 60, recommend specific actions. Do not just say "needs work."
