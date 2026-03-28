# Prompt Engineering Patterns Catalog

All patterns extracted from G-Stack, organized by category. Each pattern includes what it does, when to use it, and a concrete example.

Total: 25 patterns across 6 categories.

---

## Structural Patterns

These patterns control the shape and organization of a SKILL.md file.

### Pattern 1: Tiered Preamble Injection

**What:** A shared preamble block injected into every skill, with complexity controlled by a tier number (1-4). Each tier is additive.

**When to use:** Any repo with 2+ skills that need shared behavioral scaffolding.

**How it works:**
- T1: Core setup (upgrade check, telemetry, session tracking, trimmed voice, completion protocol)
- T2: T1 + full voice + AskUserQuestion format + Completeness Principle
- T3: T2 + repo-mode detection + Search Before Building
- T4: T3 + test failure triage + ownership classification

**Example:**
```yaml
preamble-tier: 2  # Interactive skill, needs voice + question format
```

**Why it matters:** Simple skills (browse, benchmark) get minimal overhead. Complex workflows (ship, QA) get the full behavioral stack. No skill carries scaffolding it doesn't need. Adding a new directive to T2 automatically propagates to all T2+ skills.

---

### Pattern 2: YAML Frontmatter as Trigger System

**What:** YAML frontmatter at the top of each SKILL.md that serves dual purpose: machine-readable metadata AND natural language trigger phrases for skill invocation.

**When to use:** Every SKILL.md file.

**Structure:**
```yaml
description: |
  Line 1: What it does (imperative voice)
  Line 2: "Use when asked to..." (trigger phrases)
  Line 3: "Proactively suggest when..." (auto-invoke conditions)
  Line 4: "For X, use /other instead" (disambiguation)
```

**Example:**
```yaml
description: |
  Systematic debugging with root cause investigation. Four phases:
  investigate, analyze, hypothesize, implement.
  Use when asked to "debug this", "fix this bug", "why is this broken",
  "investigate this error", or "root cause analysis".
  Proactively suggest when the user reports errors or unexpected behavior.
  For report-only mode, use /qa-only instead.
```

**Why it matters:** The description is the primary mechanism for skill discovery. Vague descriptions mean skills never get triggered. Explicit trigger phrases make invocation reliable.

---

### Pattern 3: Phase-Based Workflow Structure

**What:** Every non-trivial skill organized into numbered, sequential phases with clear boundaries.

**When to use:** Any skill that takes more than one step to complete (T2+).

**Standard sequence:**
1. Context Gathering (read code, understand state)
2. Analysis (classify, prioritize, triage)
3. Action (make changes, run commands)
4. Verification (prove it worked with evidence)
5. Report (structured output with status)

**Example:**
```markdown
## Phase 1: Context Gathering
Read the diff, check recent commits, understand the current branch state.

## Phase 2: Classification
Classify each issue as critical/high/medium/low using the severity table.

## Phase 3: Fix
Apply fixes in priority order. Commit each fix atomically.

## Phase 4: Verification
Re-run the test suite. Paste output as evidence.

## Phase 5: Report
Output structured summary with health score.
```

**Why it matters:** Without phases, agents jump to action without understanding context. They skip verification. They produce no report. Phases impose discipline.

---

### Pattern 4: Parameter Tables with Defaults

**What:** Tabular format for skill parameters, showing the parameter name, default value, and override example.

**When to use:** Any skill that accepts configurable inputs.

**Example:**
```markdown
| Parameter | Default | Override example |
|-----------|---------|-----------------|
| Target URL | (auto-detect from project) | `https://myapp.com` |
| Tier | Standard | `--quick`, `--exhaustive` |
| Browser | Chromium | `--firefox` |
```

**Why it matters:** Tables give agents a decision framework. Instead of asking the user about every parameter, the agent uses defaults and only asks when the user explicitly overrides. This eliminates unnecessary confirmation prompts.

---

### Pattern 5: Structured Output Templates

**What:** Predefined output format that every skill run must produce, ensuring consistent and parseable results.

**When to use:** Every skill that produces a report or summary.

**Example:**
```
REPORT
================================
Task:            [what was requested]
Result:          [what was done]
Evidence:        [test output, screenshots, diffs]
Status:          DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
================================
```

**Why it matters:** Consistent output means downstream skills can parse results. The ship skill can read the QA report. The retrospective can read the ship log. Without structure, outputs are unparseable prose.

---

### Pattern 6: Template Placeholder System

**What:** `{{PLACEHOLDER}}` syntax in `.tmpl` files, resolved at build time by TypeScript functions that read source code.

**When to use:** Repos with 2+ skills that share common blocks (preamble, browse setup, command references).

**How it works:**
```
SKILL.md.tmpl  -->  gen-skill-docs.ts  -->  SKILL.md
(human prose)       (resolves from code)     (committed, CI-validated)
```

**Key placeholders:**
- `{{PREAMBLE}}` -- Universal skill setup
- `{{BROWSE_SETUP}}` -- Binary discovery and daemon startup
- `{{COMMAND_REFERENCE}}` -- Auto-generated command table from code
- `{{TEST_BOOTSTRAP}}` -- Test framework detection
- `{{QA_METHODOLOGY}}` -- Shared QA phases used by /qa and /qa-only
- `{{DESIGN_METHODOLOGY}}` -- Shared design audit methodology
- `{{CO_AUTHOR_TRAILER}}` -- Git commit trailer
- `{{SNAPSHOT_FLAGS}}` -- Flag reference with examples
- `{{REVIEW_DASHBOARD}}` -- Review readiness status
- `{{BASE_BRANCH_DETECT}}` -- Dynamic main/master detection

**Why it matters:** Docs generated from code never drift. When a browse command is added in code, it appears in docs. When removed, it disappears. CI validates freshness with `gen:skill-docs --dry-run && git diff --exit-code`.

---

## Behavioral Patterns

These patterns control how the agent behaves during execution.

### Pattern 7: Iron Law / Hard Gate

**What:** A single, inviolable constraint placed before any action phases. The ONE rule that cannot be broken.

**When to use:** Every T2+ skill, especially those where scope creep is dangerous.

**Examples:**
- Investigation: "NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST"
- Brainstorming: "Do NOT invoke any implementation skill, write any code, or create any file"
- Code review: "NEVER modify code. Report only."
- Ship: "NEVER SKIP TESTS"

**Why it matters:** Without a hard gate, agents "helpfully" exceed scope. An investigation skill starts writing fixes before understanding the problem. A brainstorming skill starts coding. The Iron Law prevents this.

---

### Pattern 8: Dual Stop Condition Lists

**What:** Two explicit lists that tell the agent exactly when to stop and when to keep going.

**When to use:** Every skill that takes autonomous action (T2+).

**Example:**
```markdown
**Only stop for:**
- On the base branch (abort)
- Merge conflicts that can't be auto-resolved
- In-branch test failures after 3 fix attempts

**Never stop for:**
- Uncommitted changes (always include them)
- Version bump choice (auto-pick MICRO)
- CHANGELOG content (auto-generate from diff)
- Missing test coverage for unrelated files
```

**Why it matters:** "Only stop for" prevents the agent from halting on non-issues. "Never stop for" prevents the agent from asking trivial confirmation questions that waste the user's time. Both lists are necessary. Missing the "never stop for" list is the single most common cause of agent chattiness.

---

### Pattern 9: 3-Strike Escalation Rule

**What:** After N consecutive failures (typically 3), the agent must stop and escalate rather than continue trying.

**When to use:** Any skill that involves retry logic, debugging, or iterative fixes.

**Example:**
```markdown
If 3 hypotheses fail, STOP and report:
- What was tried
- What evidence was gathered
- Recommended next steps for the user
```

**Why it matters:** Without a strike limit, agents spin forever on hard problems, burning tokens and producing no useful output. The 3-strike rule caps waste and produces a useful handoff. "Bad work is worse than no work."

---

### Pattern 10: Repo Mode Detection (Solo vs Collaborative)

**What:** Detect whether the user is the sole contributor (fix everything) or one of many (flag issues, don't fix).

**When to use:** T3+ skills where the appropriate action depends on ownership.

**How it works:**
```bash
source <(gstack-repo-mode)
# Sets REPO_MODE=solo or REPO_MODE=collaborative
```

**Behavioral impact:**
- Solo: "Fix all issues directly. You own this code."
- Collaborative: "Flag issues in review comments. Don't modify code you don't own."

**Why it matters:** A code review skill should fix typos in a solo repo but only flag them in a collaborative one. Context changes behavior.

---

### Pattern 11: Completeness Principle with Effort Table

**What:** A directive to always do the complete thing when marginal cost is low, backed by a reference table of effort compression ratios.

**When to use:** T2+ skills where agents might recommend shortcuts.

**Example:**
```markdown
| Task type | Human team | AI-assisted | Compression |
|-----------|-----------|-------------|-------------|
| Boilerplate | 2 days | 15 min | ~100x |
| Tests | 1 day | 15 min | ~50x |
| Feature | 1 week | 30 min | ~30x |
| Bug fix | 4 hours | 15 min | ~20x |
| Architecture | 2 days | 4 hours | ~5x |
| Research | 1 day | 3 hours | ~3x |

Anti-pattern: "Choose B -- it covers 90% with less code."
If A is 70 lines more, choose A. AI makes completeness near-free.
```

**Why it matters:** Agents default to recommending the simpler option. The effort table reframes the decision: if the complete implementation costs 15 minutes of AI time, there is no reason to ship the 90% version.

---

### Pattern 12: Non-Destructive by Default

**What:** Never overwrite existing files, delete data, or make irreversible changes without explicit user approval. Show diffs first.

**When to use:** Every skill that modifies files or state.

**Example:**
```markdown
Before overwriting any file:
1. Show the diff between existing and proposed content
2. Ask for explicit approval
3. Only then write the file
```

**Why it matters:** An agent that silently overwrites a user's hand-edited config with a generated version destroys work. Non-destructive defaults protect against this.

---

## Verification Patterns

These patterns ensure work quality and prevent false completion claims.

### Pattern 13: Evidence-Based Verification

**What:** Every verification step must produce concrete evidence. Claims without proof are rejected.

**When to use:** Every skill with a verification phase.

**Example:**
```markdown
**Rationalization prevention:**
- "Should work now" -> RUN IT
- "I'm confident" -> Confidence is not evidence
- "I already tested earlier" -> Code changed since then. Test again.
```

**Why it matters:** Agents will claim success without actually verifying. They generate plausible-sounding "should work" statements. Explicit rationalization prevention forces them to actually run the check.

---

### Pattern 14: Completion Status Protocol

**What:** Four standardized completion statuses that every skill must end with, each with specific requirements.

**When to use:** Every skill.

**Statuses:**
- **DONE** -- All steps completed. Evidence attached.
- **DONE_WITH_CONCERNS** -- Completed, but specific concerns listed.
- **BLOCKED** -- Cannot proceed. Includes what was tried and what to do next.
- **NEEDS_CONTEXT** -- Missing specific information. States exactly what is needed.

**Escalation rules:**
- 3 failed attempts -> STOP and escalate
- Uncertain about security -> STOP and escalate
- "Bad work is worse than no work"

**Why it matters:** Unstructured endings like "I think that's done" give the user no signal about confidence or completeness. Structured statuses make the outcome unambiguous.

---

### Pattern 15: Before/After Scoring

**What:** Show measurable improvement with numeric scores before and after the skill's work.

**When to use:** QA, design review, code review, and any skill that improves a measurable quality.

**Example:**
```markdown
Health Score: 45/100 -> 87/100 (+42 points)

| Category | Before | After | Change |
|----------|--------|-------|--------|
| Critical bugs | 3 | 0 | -3 |
| High bugs | 5 | 1 | -4 |
| Medium bugs | 8 | 3 | -5 |
```

**Why it matters:** Without numbers, "I fixed some bugs" is meaningless. Quantified improvement shows the actual impact and helps the user decide if the work is done or needs another pass.

---

### Pattern 16: Blame Protocol

**What:** When claiming an issue is pre-existing (not caused by current changes), require proof. No attribution without receipts.

**When to use:** Code review, QA, and ship skills that encounter failing tests.

**Example:**
```markdown
"Pre-existing without receipts is a lazy claim. Prove it or don't say it."

To classify a test failure as pre-existing:
1. Check if the test passes on the base branch
2. If yes, the failure is in-branch (you own it, fix it)
3. If no, the failure is pre-existing (document it, don't block on it)
```

**Why it matters:** Agents love to blame pre-existing issues to avoid doing work. Requiring proof prevents false attribution and ensures real failures get fixed.

---

### Pattern 17: Atomic Commit Verification

**What:** Each fix is committed individually and verified before moving to the next.

**When to use:** QA and bug-fix skills where multiple issues are addressed.

**Example:**
```markdown
For each bug found:
1. Fix the bug in source code
2. Commit with descriptive message
3. Re-run verification for that specific fix
4. Take evidence (screenshot, test output)
5. Only then move to the next bug
```

**Why it matters:** Batching all fixes into one commit makes it impossible to bisect which fix broke what. Atomic commits with per-fix verification create a clean, reversible history.

---

## Voice Patterns

These patterns control the tone, style, and personality of agent output.

### Pattern 18: Persona-Based Voice with Banned Vocabulary

**What:** A voice directive that defines tone through a persona model AND explicitly bans AI slop vocabulary.

**When to use:** T1+ (trimmed voice for simple skills, full voice for T2+).

**Example:**
```markdown
**Tone:** direct, concrete, sharp, never corporate, never academic.
Sound like a builder, not a consultant.

**Banned words:** delve, crucial, robust, comprehensive, nuanced, multifaceted,
furthermore, moreover, additionally, pivotal, landscape, tapestry, underscore,
foster, showcase, intricate, vibrant, fundamental, significant, interplay.

**Banned phrases:** "here's the kicker", "let me break this down",
"at the end of the day", "here's the thing", "plot twist",
"the bottom line", "make no mistake", "can't stress this enough".

**Writing rules:**
- No em dashes (use commas, periods, "...")
- Short paragraphs. Mix one-sentence with 2-3 sentence runs.
- Name the file, the function, the line number.
- End with what to do. Give the action.
- Dry humor about the absurdity of software is fine.

**Final test:** Does this sound like a real builder talking to a builder?
```

**Why it matters:** Without a banned vocabulary list, AI output defaults to corporate consultant prose. Banning specific words forces concrete, direct communication.

---

### Pattern 19: Context-Sensitive Tone Calibration

**What:** Different energy levels for different contexts within the same skill.

**When to use:** Skills that operate in multiple modes (strategy vs execution).

**Example:**
```markdown
YC partner energy for strategy discussions.
Senior eng energy for code review.
Never consultant energy. Never academic energy.
```

**Why it matters:** A single tone doesn't work everywhere. Strategy conversations need high-energy challenge. Code review needs precise, measured analysis. Calibrating tone to context produces better output.

---

## Orchestration Patterns

These patterns control how skills interact with each other and with the user.

### Pattern 20: AskUserQuestion Format

**What:** Every user-facing question follows a 4-part format that re-grounds context, simplifies the question, recommends an option, and provides lettered choices with effort estimates.

**When to use:** Every skill that asks the user a question (T2+).

**Example:**
```markdown
1. **Re-ground:** "In project gstack, on branch feat/qa-improvements,
   fixing the snapshot timeout issue..."
2. **Simplify:** "The timeout happens because the browser takes too long to
   render. We can either increase the timeout or optimize the rendering."
3. **Recommend:** "RECOMMENDATION: Choose A because it's a 5-minute fix
   that solves the root cause. Completeness: 9/10"
4. **Options:**
   A) Increase timeout to 10s (human: ~0 / AI: ~2 min)
   B) Optimize rendering pipeline (human: ~0 / AI: ~30 min)
   C) Skip for now, add to TODOS (human: ~0 / AI: ~1 min)
```

**Why it matters:** Without re-grounding, users who haven't looked at the session in 20 minutes are lost. Without recommendations, users face decision fatigue. Without effort estimates, they can't make informed tradeoffs.

---

### Pattern 21: Cross-Skill Reference Graph

**What:** Each skill explicitly references the skills that come before and after it in the development workflow.

**When to use:** Any repo with 3+ skills that form a pipeline.

**Example:**
```markdown
Suggest adjacent skills when relevant:
- Brainstorming -> /office-hours
- Strategy -> /plan-ceo-review
- Architecture -> /plan-eng-review
- Design -> /plan-design-review or /design-consultation
- Implementation -> [user writes code]
- Review -> /review
- QA -> /qa
- Ship -> /ship
- Docs -> /document-release
- Deploy -> /land-and-deploy
- Monitor -> /canary
- Retro -> /retro
```

**Why it matters:** Without cross-references, skills are isolated islands. The user finishes a code review and doesn't know that /ship exists. The workflow graph connects the lifecycle.

---

### Pattern 22: Auto-Invocation Chaining

**What:** Skills that automatically invoke other skills as sub-steps within their workflow.

**When to use:** Complex workflows where manual chaining adds friction.

**Example:**
```markdown
/ship automatically runs:
- Step 3.5: /review (pre-landing code review)
- Step 8.5: /document-release (post-ship doc sync)

/autoplan chains:
- /plan-ceo-review -> /plan-design-review -> /plan-eng-review
```

**Why it matters:** Manual chaining means the user must remember to run /review before /ship. Auto-invocation enforces the correct sequence and eliminates forgotten steps.

---

### Pattern 23: Cross-Model Second Opinion

**What:** A skill that invokes a different AI model to provide an independent review.

**When to use:** Code review, architecture decisions, or any high-stakes output.

**Example:**
```markdown
/codex provides three modes:
- Code review: Independent diff review with pass/fail gate
- Challenge: Adversarial testing ("try to break this")
- Consult: Ask anything with session continuity for follow-ups
```

**Why it matters:** Same-model review has blind spots. A different model catches different issues. The cross-model pattern is the AI equivalent of peer review.

---

## Infrastructure Patterns

These patterns support the skill system itself.

### Pattern 24: Persistent State via Daemon

**What:** A long-lived background process (browse daemon) that maintains state between commands instead of cold-starting each time.

**When to use:** Any skill that needs fast, repeated access to an external resource (browser, database, API).

**How it works:**
- First call: ~3s startup (launch browser, load page)
- Subsequent calls: ~100-200ms (reuse existing browser session)
- Cookies, tabs, localStorage persist between commands
- Auto-shutdown after idle timeout (30 min)
- Security: localhost-only, bearer token auth

**Why it matters:** Cold-starting a browser for every command adds 3 seconds per call. Over a QA session with 50+ browser commands, that is 2.5 minutes of wait time eliminated.

---

### Pattern 25: Session Tracking and Multi-Session Awareness

**What:** Track active sessions by PID. When 3+ sessions are active, re-ground every question with full context because the user is context-switching.

**When to use:** Any repo where the user might run multiple agent sessions in parallel.

**How it works:**
```bash
# Session tracking
mkdir -p ~/.gstack/sessions
touch ~/.gstack/sessions/"$PPID"
_SESSIONS=$(find ~/.gstack/sessions -mmin -120 -type f | wc -l)
```

**Behavioral impact:**
- 1 session: Normal question format
- 3+ sessions: Extra context in every question (user is juggling)

**Why it matters:** When a user has 3 agent sessions running, they can't remember what each one is doing. Extra re-grounding prevents confusion and wrong decisions.

---

## Summary by Category

| Category | Patterns | Count | Purpose |
|----------|----------|-------|---------|
| Structural | 1-6 | 6 | Control file shape and organization |
| Behavioral | 7-12 | 6 | Control agent execution behavior |
| Verification | 13-17 | 5 | Ensure work quality and prevent false claims |
| Voice | 18-19 | 2 | Control output tone and style |
| Orchestration | 20-23 | 4 | Control skill interaction and user communication |
| Infrastructure | 24-25 | 2 | Support the skill system itself |
| **Total** | | **25** | |
