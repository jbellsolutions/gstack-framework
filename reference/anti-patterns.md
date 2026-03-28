# Anti-Patterns: What NOT to Do

10 patterns that make skills, agents, and repos worse. Each entry covers what it is, why it is bad, what to do instead, and a concrete example showing the wrong way vs the right way.

---

## 1. Hand-Writing Generated Content

**What it is:** Writing SKILL.md files by hand when they reference information that exists in source code (commands, flags, paths, configuration).

**Why it is bad:** Docs drift from code within days. A command gets renamed, a flag gets added, the doc says one thing and the code does another. The agent follows the doc, hits an error, and wastes a cycle trying to recover. Multiply this by every skill and you have systematic unreliability.

**What to do instead:** Use a template system. Write `.tmpl` files with `{{PLACEHOLDER}}` syntax. Build a compiler that resolves placeholders from source code. Commit the generated output. Add CI validation: `gen:skill-docs --dry-run && git diff --exit-code`. Docs become code artifacts, not hand-written prose.

**Example:**

Wrong:
```markdown
## Available Commands
- `$B navigate <url>` -- Open a URL
- `$B click <selector>` -- Click an element
- `$B screenshot` -- Take a screenshot
```
(This list was copy-pasted. When `$B type` is added to the code, nobody updates the doc.)

Right:
```markdown
## Available Commands
{{COMMAND_REFERENCE}}
```
(The placeholder resolves from the actual command registry. New commands appear automatically. Removed commands disappear. CI catches drift.)

---

## 2. No Stop Conditions

**What it is:** A skill that does not define when the agent should stop autonomously vs when it should ask the user.

**Why it is bad:** The agent asks for trivial confirmations constantly: "Ready to push?" "Should I commit?" "Shall I proceed with the version bump?" "Which format for the changelog?" Each question breaks the user's flow, adds latency, and provides zero value. The worst case: the agent stops on every single step, turning an autonomous workflow into a chatbot.

**What to do instead:** Define two explicit lists. "Only stop for" covers genuine blockers where user input is required. "Never stop for" covers the trivial decisions the agent should make on its own with sensible defaults.

**Example:**

Wrong:
```markdown
## Phase 3: Ship
Push the code and create a PR.
```
(No guidance on when to stop. Agent asks: "Should I push?" "What branch?" "Include uncommitted changes?" "What version bump?" "Write a changelog entry?")

Right:
```markdown
## Phase 3: Ship

**Only stop for:**
- On the base branch (abort, wrong branch)
- Merge conflicts that can't be auto-resolved
- In-branch test failures after 3 fix attempts
- Security-sensitive changes detected

**Never stop for:**
- Uncommitted changes (always include them)
- Version bump choice (auto-pick MICRO unless breaking change)
- CHANGELOG content (auto-generate from diff)
- Commit message format (use conventional commits)
- PR description (auto-generate from changelog)
```

---

## 3. Generic Voice (No AI Slop Prevention)

**What it is:** No voice directive, or a vague one like "be helpful and professional."

**Why it is bad:** Without explicit vocabulary bans, agent output defaults to the AI slop register: "Let's delve into the robust landscape of comprehensive solutions that fundamentally showcase the intricate tapestry of modern software engineering." This tone erodes user trust. It sounds fake. It wastes tokens on filler. It buries the actual information.

**What to do instead:** Ban specific words by name. Define a persona. Provide a final test ("does this sound like a real builder talking to a builder?"). Include writing rules about paragraph length, em dashes, and specificity.

**Example:**

Wrong:
```markdown
## Voice
Be helpful, professional, and thorough in your responses.
```
(Agent output: "I've comprehensively analyzed the codebase and identified several crucial areas that fundamentally need attention.")

Right:
```markdown
## Voice
Tone: direct, concrete, sharp, never corporate, never academic.
Sound like a builder, not a consultant. Name the file, the function, the command.

Banned words: delve, crucial, robust, comprehensive, nuanced, multifaceted,
furthermore, moreover, additionally, pivotal, landscape, tapestry, underscore,
foster, showcase, intricate, vibrant, fundamental, significant, interplay.

Writing rules:
- No em dashes. Use commas, periods, or "..." instead.
- Short paragraphs. End with what to do.
- Name the file, the function, the line number.

Final test: Does this sound like a real builder talking to a builder?
```
(Agent output: "Found a null pointer in `browse/src/server.ts:142`. The `tabId` param is optional but used without a check. Fix: add a guard clause. 3 lines.")

---

## 4. No Verification Step

**What it is:** A skill that claims success without requiring proof that the work actually succeeded.

**Why it is bad:** The agent generates a fix, says "this should resolve the issue," and moves on. But the fix has a typo. Or it fixed the wrong file. Or the test still fails. Unverified claims ship broken code. The user trusts the agent's confidence and merges without checking. Now the bug is in production.

**What to do instead:** Add an explicit verification phase. Require the agent to re-run the relevant test, paste the output, and confirm no regressions. Include rationalization prevention rules that reject claims without evidence.

**Example:**

Wrong:
```markdown
## Phase 3: Fix the Bug
1. Identify the root cause
2. Apply the fix
3. Done!
```
(Agent: "I've updated the handler to check for null. This should fix the crash." It didn't. The crash still happens.)

Right:
```markdown
## Phase 4: Verification

IRON LAW: NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE.

1. Re-run the failing test: `bun test browse/src/server.test.ts`
2. Paste the full output
3. Confirm zero failures
4. If verification fails -> fix and re-verify. Do NOT skip.

Rationalization prevention:
- "Should work now" -> RUN IT
- "I'm confident" -> Confidence is not evidence
- "I already tested earlier" -> Code changed since then. Test again.
```

---

## 5. No Escalation Path

**What it is:** A skill with no mechanism for the agent to say "I'm stuck. I can't solve this. Here is what I tried."

**Why it is bad:** The agent tries the same approach repeatedly. Or it tries increasingly creative (and increasingly wrong) approaches. Or it silently skips the hard part and reports success on the easy parts. Token burn adds up. The user gets a garbage result 20 minutes later. Worst case: the agent makes things worse by applying wrong fixes.

**What to do instead:** Implement a 3-strike rule. After 3 failed attempts at the same task, the agent must stop and produce a structured handoff: what was tried, what evidence was gathered, and recommended next steps. "Bad work is worse than no work."

**Example:**

Wrong:
```markdown
## Phase 3: Debug
Find and fix the issue.
```
(Agent tries approach 1, fails. Tries approach 2, fails. Tries approach 3, fails. Tries approach 4... 20 minutes later, still spinning.)

Right:
```markdown
## Phase 3: Debug

For each hypothesis:
1. State the hypothesis
2. Design a test for it
3. Run the test
4. Record the result

If 3 hypotheses fail, STOP and report:
- Hypotheses tested with results
- Evidence gathered
- Recommended next steps for the user
- Status: BLOCKED

Do NOT keep trying past 3 strikes. Escalate.
```

---

## 6. Monolithic Preamble

**What it is:** Every skill gets the same 2000-word preamble regardless of whether it needs test failure triage, repo ownership detection, contributor mode, search-before-building directives, and the full AskUserQuestion specification.

**Why it is bad:** Context tokens are finite. A simple browse skill that takes screenshots doesn't need test failure triage logic. Loading 2000 words of behavioral scaffolding into a T1 skill wastes context window on instructions the agent will never use, potentially pushing out the actual task content that matters.

**What to do instead:** Tier the preamble. T1 gets the minimum (voice, completion protocol). T2 adds the question format and completeness principle. T3 adds repo mode and search directives. T4 adds test failure triage. Each skill declares its tier and gets only what it needs.

**Example:**

Wrong:
```markdown
# Every skill, including "take a screenshot":
[2000 words of preamble including test failure triage, repo mode detection,
 search-before-building, contributor mode, AskUserQuestion format,
 completeness principle, effort compression table...]
```

Right:
```yaml
# Simple browse skill:
preamble-tier: 1  # Gets: voice (trimmed), completion protocol, upgrade check

# Complex ship workflow:
preamble-tier: 4  # Gets: everything from T1-T3 + test failure triage
```

---

## 7. No Cross-Skill References

**What it is:** Skills that exist as isolated islands with no awareness of each other or the broader development workflow.

**Why it is bad:** The user finishes a QA pass and asks "what next?" The agent doesn't know that /ship exists. The user finishes /ship and doesn't know about /document-release. Every transition between workflow stages requires the user to manually remember which skill to invoke next. This defeats the purpose of having a skill system.

**What to do instead:** Build a workflow graph. Each skill should know its neighbors: what comes before it and what comes after it. Suggest the next skill when the current one completes. For tight couplings, use auto-invocation (Pattern 22).

**Example:**

Wrong:
```markdown
# /qa skill
## Phase 5: Report
Here is the QA summary.
Status: DONE
```
(User: "Great, now what?" Agent: "I don't know.")

Right:
```markdown
# /qa skill
## Phase 5: Report
Here is the QA summary.
Status: DONE

Suggest adjacent skills:
- If bugs were found and fixed: run /review to check the fixes
- If all clean: run /ship to push and create PR
- If design issues found: run /design-review for visual polish
- For a second opinion on the fixes: run /codex review
```

---

## 8. Testing Only Code, Not Prompts

**What it is:** A test suite that covers the source code (unit tests, integration tests) but never tests whether the SKILL.md files actually produce correct agent behavior.

**Why it is bad:** A one-line change to a preamble can break every skill that uses it. "Added a sentence about being thorough" might cause the agent to skip verification because it now considers thoroughness sufficient. Prompt changes are as dangerous as code changes, but most repos test only code.

**What to do instead:** Implement three-tier testing. Tier 1 (free, fast): parse every command reference in SKILL.md, validate it exists in the registry. Tier 2 (expensive, slow): spawn real agent sessions, run skills end-to-end, check for errors. Tier 3 (cheap, medium): use an LLM-as-judge to score skill output quality.

**Example:**

Wrong:
```
test/
  server.test.ts      # Tests the browse server code
  commands.test.ts     # Tests command parsing
  # No skill-level tests at all
```

Right:
```
test/
  server.test.ts               # Code tests (as before)
  commands.test.ts              # Code tests (as before)
  skill-validation.test.ts     # Tier 1: parse SKILL.md, validate command refs
  skill-e2e-qa.test.ts         # Tier 2: spawn Claude, run /qa, check result
  skill-llm-eval.test.ts       # Tier 3: Sonnet scores doc quality 0-10
```

---

## 9. Vague Descriptions

**What it is:** Skill descriptions that are generic, human-readable summaries but don't include the specific phrases a user or agent would say to trigger the skill.

**Why it is bad:** The skill discovery system matches user input against descriptions. If the description is "A testing tool," it won't trigger when the user says "does this work?", "check for bugs", "QA this", or "find what's broken." The skill exists but never gets invoked. It's invisible.

**What to do instead:** Treat the description as a trigger specification. Line 1: what it does. Line 2: explicit trigger phrases (every way a user might ask for this). Line 3: when to proactively suggest it. Line 4: when NOT to use it (disambiguation from similar skills).

**Example:**

Wrong:
```yaml
description: A testing and quality assurance tool.
```
(User says "does this work?" -- skill not triggered. User says "find bugs" -- skill not triggered.)

Right:
```yaml
description: |
  Systematically QA test a web application and fix bugs found.
  Use when asked to "qa", "QA", "test this site", "find bugs",
  "test and fix", "fix what's broken", or "does this work".
  Proactively suggest when the user says a feature is ready for testing.
  For report-only mode (no fixes), use /qa-only instead.
```

---

## 10. No Iron Law

**What it is:** A skill without a single, inviolable core constraint that defines what the agent must NEVER do.

**Why it is bad:** Agents are eager to help. Without a hard boundary, they exceed scope in predictable ways. A brainstorming skill starts writing code. A debugging skill starts refactoring. A code review skill starts implementing fixes. A QA skill starts adding features. The user asked for one thing and got something bigger, messier, and harder to undo.

**What to do instead:** Define the ONE rule that cannot be violated. Place it prominently before any action phases. Make it specific and testable. The Iron Law should prevent the single most likely mode of scope creep for that skill's domain.

**Example:**

Wrong:
```markdown
# Investigate Skill
Debug the issue the user reports.

## Phase 1: Look at the code
...
```
(Agent finds a bug, immediately starts writing a fix, introduces a new bug in the fix, spends 15 minutes debugging its own fix instead of the original issue.)

Right:
```markdown
# Investigate Skill

## Iron Law

**NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST.**

Do not write a single line of fix code until:
1. The root cause is identified with evidence
2. The hypothesis is stated explicitly
3. The user approves the diagnosis

Only then proceed to Phase 3 (Implementation).
```

---

## Quick Reference

| # | Anti-Pattern | One-Line Fix |
|---|-------------|-------------|
| 1 | Hand-writing generated content | Use templates + CI validation |
| 2 | No stop conditions | Add "only stop for" and "never stop for" lists |
| 3 | Generic voice | Ban specific AI slop words by name |
| 4 | No verification step | Require evidence, prevent rationalization |
| 5 | No escalation path | 3-strike rule with structured handoff |
| 6 | Monolithic preamble | Tier preambles T1-T4 by complexity |
| 7 | No cross-skill references | Build a workflow graph, suggest next skill |
| 8 | Testing only code, not prompts | Add 3-tier skill testing (static, E2E, LLM judge) |
| 9 | Vague descriptions | Include explicit trigger phrases and disambiguation |
| 10 | No Iron Law | Define the ONE inviolable constraint per skill |
