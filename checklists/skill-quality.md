# SKILL.md Quality Checklist

Evaluate any individual SKILL.md file against these criteria. Each section includes what to check, what good looks like, and common failures.

---

## 1. Frontmatter Completeness

### Required Fields

- [ ] **`name`** -- Kebab-case skill name matching the directory name
  - Good: `name: investigate`
  - Bad: `name: Investigation Skill` (spaces, capitalization)

- [ ] **`description`** -- Multi-line with structured trigger information
  - Line 1: What it does in imperative voice
  - Line 2: Trigger phrases ("Use when asked to...")
  - Line 3: Proactive trigger ("Proactively suggest when...")
  - Line 4: Disambiguation ("For X, use /other instead")
  - Good:
    ```yaml
    description: |
      Systematic debugging with root cause investigation.
      Use when asked to "debug this", "fix this bug", "why is this broken".
      Proactively suggest when the user reports errors or unexpected behavior.
      For report-only mode, use /qa-only instead.
    ```
  - Bad: `description: A debugging tool` (single line, no triggers)

- [ ] **`allowed-tools`** -- Explicit tool whitelist, minimal and justified
  - Good: Design-only skill omits Write/Edit tools
  - Bad: Every skill lists every tool available

- [ ] **`preamble-tier`** -- Set to 1-4, matched to skill complexity
  - T1: Simple read-only or browser skills
  - T2: Interactive skills needing voice + question format
  - T3: Skills that search, research, or evaluate
  - T4: Ship/QA/review skills with test failure handling

### Optional Fields

- [ ] **`version`** -- Semver string present
- [ ] **`benefits-from`** -- Lists complementary skills
- [ ] **`hooks`** -- Pre-tool-use hooks if the skill needs guardrails (freeze, guard)

---

## 2. Iron Law / Hard Gate

- [ ] **Present** -- Every complex skill (T2+) has an Iron Law or Hard Gate section
- [ ] **Specific** -- States the ONE constraint that cannot be violated
  - Good: "NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST"
  - Good: "Do NOT invoke any implementation skill, write any code, or create any file"
  - Bad: "Be careful" (vague, unenforceable)
- [ ] **Positioned early** -- Appears before any action phases, not buried at the bottom
- [ ] **Prevents scope creep** -- Stops the agent from "helpfully" doing more than asked
  - Good: Office-hours gates out of implementation. Investigate gates out of premature fixes.
  - Bad: No gate, agent writes code during a brainstorming session

---

## 3. Phase-Based Workflow

- [ ] **Numbered phases** -- Phase 1, Phase 2, etc. with descriptive names
- [ ] **Standard pattern followed** -- Context -> Analysis -> Action -> Verification -> Report
- [ ] **Clear inputs per phase** -- What the phase reads, checks, or receives
  - Good: "Read the diff with `git diff main...HEAD`"
  - Bad: "Look at the code" (which code? how?)
- [ ] **Clear outputs per phase** -- What the phase produces or changes
  - Good: "Produce a prioritized list of issues classified as critical/high/medium/low"
  - Bad: "Analyze things" (no deliverable)
- [ ] **Phase transitions are explicit** -- When to move to the next phase
  - Good: "Once all issues are classified, proceed to Phase 3"
  - Bad: Phases bleed into each other with no boundary
- [ ] **Sub-steps within phases** -- Complex phases use 3a, 3b, 3c numbering
- [ ] **No phase skipping** -- Phases are sequential, not optional

---

## 4. Verification Step

- [ ] **Verification phase exists** -- Explicitly named, not just implied
- [ ] **Evidence requirement** -- Must produce proof, not just claim success
  - Good: "Re-run the test suite and paste the output"
  - Bad: "Make sure it works"
- [ ] **Fresh evidence** -- Cannot reuse old test results
  - Good: "Code changed since then. Test again."
  - Bad: "I already tested earlier" accepted as sufficient
- [ ] **Rationalization prevention** -- Explicit rules against skipping verification
  - Required phrases:
    - "Should work now" -> RUN IT
    - "I'm confident" -> Confidence is not evidence
    - "I already tested earlier" -> Code changed since then. Test again.
- [ ] **Failed verification triggers rework** -- Not skip, not ignore, but fix and re-verify

---

## 5. Stop Conditions

### "Only stop for" list
- [ ] **Present** -- Explicit list of conditions that warrant stopping
- [ ] **Specific** -- Each condition is concrete and testable
  - Good: "On the base branch (abort)", "Merge conflicts that can't be auto-resolved"
  - Bad: "When something goes wrong"
- [ ] **Covers safety** -- Includes security uncertainty, destructive operations

### "Never stop for" list
- [ ] **Present** -- Explicit list of trivial things agents waste time asking about
- [ ] **Covers common agent hesitations** -- Things agents normally pause for unnecessarily
  - Good: "Uncommitted changes (always include them)", "Version bump choice (auto-pick MICRO)"
  - Bad: Missing entirely (agents will ask 10 unnecessary questions)
- [ ] **Balanced** -- Neither too restrictive (agent stops for everything) nor too permissive (agent never stops)

---

## 6. AskUserQuestion Format Compliance

- [ ] **Format specified** -- 4-part structure is documented in the skill
- [ ] **Re-ground** -- Step 1 states project, branch, current task (assumes user hasn't looked in 20 min)
- [ ] **Simplify** -- Step 2 uses plain English a 16-year-old could follow
- [ ] **Recommend** -- Step 3 states recommendation with reason and Completeness score (X/10)
- [ ] **Options** -- Step 4 provides lettered options with effort estimates (human: ~X / AI: ~Y)
- [ ] **Used consistently** -- Every user-facing question in the skill follows this format

---

## 7. Voice Directive

- [ ] **Voice section exists** -- Tone definition present
- [ ] **Tone is defined** -- "direct, concrete, sharp, never corporate, never academic"
- [ ] **Builder persona** -- "Sound like a builder, not a consultant"
- [ ] **Banned vocabulary list** -- Specific AI slop words banned:
  - delve, crucial, robust, comprehensive, nuanced, multifaceted, furthermore,
    moreover, additionally, pivotal, landscape, tapestry, underscore, foster,
    showcase, intricate, vibrant, fundamental, significant, interplay
- [ ] **Banned phrases list** -- Specific filler phrases banned:
  - "here's the kicker", "let me break this down", "at the end of the day"
- [ ] **Writing rules** -- Specific formatting rules:
  - No em dashes (use commas, periods, "...")
  - Short paragraphs
  - Name the file, the function, the line number
  - End with what to do
- [ ] **Final test** -- "Does this sound like a real builder talking to a builder?"

---

## 8. Completion Status Protocol

- [ ] **Four statuses defined** -- DONE, DONE_WITH_CONCERNS, BLOCKED, NEEDS_CONTEXT
- [ ] **DONE requires evidence** -- Not just "finished" but proof of completion
- [ ] **DONE_WITH_CONCERNS lists the concerns** -- Specific, not vague worries
- [ ] **BLOCKED explains what was tried** -- Not just "can't do it" but what failed and what to try next
- [ ] **NEEDS_CONTEXT specifies exactly what** -- Not "more info needed" but the precise missing piece
- [ ] **Escalation rules present**:
  - 3 failed attempts -> STOP and escalate
  - Uncertain about security -> STOP and escalate
  - "Bad work is worse than no work"

---

## 9. No AI Slop Vocabulary

Scan the entire SKILL.md for these terms. Zero tolerance in output instructions. Acceptable only when explicitly banning them.

### Banned Words
- [ ] No use of: delve, crucial, robust, comprehensive, nuanced, multifaceted
- [ ] No use of: furthermore, moreover, additionally, pivotal, landscape
- [ ] No use of: tapestry, underscore, foster, showcase, intricate
- [ ] No use of: vibrant, fundamental, significant, interplay
- [ ] No use of: leverage (as verb), synergy, paradigm, holistic, ecosystem

### Banned Phrases
- [ ] No use of: "here's the kicker", "let me break this down"
- [ ] No use of: "at the end of the day", "it's worth noting"
- [ ] No use of: "in today's fast-paced world", "game-changer"
- [ ] No use of: "deep dive", "unpack this", "circle back"
- [ ] No use of: "low-hanging fruit", "move the needle"

### Banned Patterns
- [ ] No exclamation marks in technical content (enthusiasm is not engineering)
- [ ] No emoji in skill instructions (save them for user-facing output if at all)
- [ ] No rhetorical questions ("But what about...?")

---

## 10. Anti-Pattern Avoidance

- [ ] **No monolithic preamble** -- Preamble is tiered, not one-size-fits-all
- [ ] **No hand-written generated content** -- If docs reference code, they are generated from code
- [ ] **No vague descriptions** -- Descriptions double as triggers with explicit phrases
- [ ] **No missing cross-skill references** -- Skill knows its neighbors in the workflow
- [ ] **No implicit verification** -- Verification is explicit, not assumed
- [ ] **No shortcut recommendations** -- If the complete thing costs minutes, do the complete thing
- [ ] **No bare assertions** -- Claims are backed by evidence, not confidence
- [ ] **No unbounded iteration** -- 3-strike rule or equivalent prevents infinite loops
- [ ] **No security bypass** -- Security uncertainty always escalates to user
- [ ] **No scope assumptions** -- Parameters have defaults but support overrides via table

---

## Quick Score

Count the checked boxes above. Each checkbox is worth 1 point.

| Checked | Rating |
|---------|--------|
| 55+ | Reference quality SKILL.md |
| 45-54 | Production grade |
| 35-44 | Ship ready |
| 25-34 | Needs work |
| <25 | Rewrite recommended |

Total checkboxes: ~60
