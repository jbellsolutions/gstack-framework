# Audit Scoring Rubric

Detailed criteria for scoring each category in repo-quality.md. For every item, this document explains what earns 0 points, partial points, and full points.

---

## Category 1: Documentation (25 points)

### ETHOS.md [5 pts]

**Existence [2 pts]**
- 0 pts: No ETHOS.md or equivalent philosophy document exists
- 1 pt: File exists but is a stub, placeholder, or generic mission statement
- 2 pts: File exists with substantive builder philosophy that shapes agent behavior

**Completeness Principle [1 pt]**
- 0 pts: No mention of completeness, thoroughness, or marginal cost reasoning
- 1 pt: Contains a clear principle about when to do the complete thing vs when to flag scope. Uses concrete framing like "lakes vs oceans" or equivalent metaphor that makes the decision boundary obvious.

**Search Before Building [1 pt]**
- 0 pts: No research directive
- 1 pt: Contains a layered knowledge model (tried-and-true, new-and-popular, first-principles) or equivalent structured approach to deciding when to reuse vs build.

**Specificity Principle [1 pt]**
- 0 pts: No guidance on building for real problems vs hypothetical ones
- 1 pt: Contains a directive like "Build for Yourself" that prioritizes specific real problems over generic hypothetical solutions.

---

### CLAUDE.md [5 pts]

**Existence [1 pt]**
- 0 pts: No CLAUDE.md
- 1 pt: File exists at repo root

**Dev Commands [1 pt]**
- 0 pts: No commands listed, or commands are wrong/outdated
- 1 pt: All essential commands (install, test, build) listed with exact syntax that can be copy-pasted into a terminal

**Project Structure [1 pt]**
- 0 pts: No structure documentation
- 1 pt: Directory tree showing the layout with brief descriptions of what lives where

**Conventions [1 pt]**
- 0 pts: No conventions documented
- 1 pt: Key patterns and naming conventions are listed (file naming, code style, commit format, etc.)

**Effort Compression [1 pt]**
- 0 pts: No guidance on AI-assisted work estimation
- 1 pt: Table or equivalent showing task types with human vs AI time estimates and compression ratios

---

### ARCHITECTURE.md [5 pts]

**Existence [1 pt]**
- 0 pts: No architecture document
- 1 pt: File exists at repo root

**Core Idea [1 pt]**
- 0 pts: No problem statement or key insight
- 1 pt: One paragraph that a new developer can read to understand WHAT problem this solves and the KEY INSIGHT behind the approach

**Technology Justification [1 pt]**
- 0 pts: No justification, or "we use X because it's popular"
- 1 pt: Concrete, numbered reasons for technology choices. Each reason explains WHY this choice over alternatives.

**Intentional Omissions [1 pt]**
- 0 pts: No documentation of what was left out
- 1 pt: Explicit list of things NOT built, with concrete reasons. This prevents future contributors from re-proposing already-rejected ideas.

**Error Philosophy [1 pt]**
- 0 pts: No error handling guidance
- 1 pt: Clear directive that errors must be actionable (include recovery suggestions, not just failure messages). Bonus: examples of bad vs good error messages.

---

### README.md [4 pts]

**Existence [1 pt]**
- 0 pts: No README
- 1 pt: README exists with a project description that explains what this is in 1-2 sentences

**Installation [1 pt]**
- 0 pts: No install instructions, or instructions that fail
- 1 pt: Working installation instructions. One-command setup preferred (`./setup` or `curl ... | bash`).

**Usage Examples [1 pt]**
- 0 pts: No examples
- 1 pt: At least one concrete usage example showing the primary workflow

**License and Contribution [1 pt]**
- 0 pts: No license, no contribution info
- 1 pt: License identified and contribution path described (even if just "see CONTRIBUTING.md")

---

### CHANGELOG.md + VERSION [3 pts]

**CHANGELOG.md [1 pt]**
- 0 pts: No changelog, or empty file
- 1 pt: Changelog exists with dated entries describing user-facing changes. Written in human-readable prose, not just commit hashes.

**VERSION [1 pt]**
- 0 pts: No version file, or version only in package.json
- 1 pt: VERSION file at repo root with semver format. 4-digit (MAJOR.MINOR.PATCH.MICRO) preferred, 3-digit (MAJOR.MINOR.PATCH) acceptable.

**Consistency [1 pt]**
- 0 pts: VERSION and CHANGELOG are out of sync or contradictory
- 1 pt: Latest CHANGELOG entry matches the VERSION number. Version bumps and changelog entries are paired chronologically.

---

### TODOS.md [3 pts]

**Existence [1 pt]**
- 0 pts: No TODO tracking, or scattered TODOs only in code comments
- 1 pt: Centralized TODOS.md exists at repo root

**Prioritization [1 pt]**
- 0 pts: Flat list with no priority indication
- 1 pt: Items categorized by priority (P0-P4, High/Medium/Low, or equivalent). P0 items are clearly urgent.

**Actionability [1 pt]**
- 0 pts: Vague items like "improve performance" or "fix bugs"
- 1 pt: Each item is a concrete next step with enough context to act on. Example: "Add timeout handling to browse daemon HTTP calls (currently hangs on unresponsive servers)"

---

## Category 2: Skill Quality (25 points)

Score the AVERAGE quality across all skills. If the repo has 10 skills and 8 are excellent but 2 are poor, the average should reflect that.

### Frontmatter [6 pts]

**name field [1 pt]**
- 0 pts: Missing, or doesn't match directory name
- 1 pt: Present, kebab-case, matches the skill directory

**description with triggers [1 pt]**
- 0 pts: Single-line generic description like "A useful tool"
- 1 pt: Multi-line description where line 2 contains explicit trigger phrases ("Use when asked to...")

**Proactive triggers [1 pt]**
- 0 pts: No proactive trigger condition
- 1 pt: Line 3 contains "Proactively suggest when..." with specific conditions

**Disambiguation [1 pt]**
- 0 pts: No disambiguation (skills with overlapping scope have no guidance)
- 1 pt: "For X, use /other-skill instead" present where skills could be confused

**allowed-tools [1 pt]**
- 0 pts: Missing (agent has unrestricted tool access) or listing every tool
- 1 pt: Explicit whitelist that is minimal and justified. Read-only skills omit Write/Edit.

**preamble-tier [1 pt]**
- 0 pts: Missing or inappropriate (T4 on a simple browse skill, T1 on a complex ship workflow)
- 1 pt: Set and appropriate. Simple skills use T1-T2, complex workflows use T3-T4.

### Iron Law / Hard Gate [3 pts]

**Presence [1 pt]**
- 0 pts: No Iron Law section in any T2+ skill
- 1 pt: Every T2+ skill has an Iron Law or Hard Gate

**Specificity [1 pt]**
- 0 pts: Vague constraint like "be careful" or "follow best practices"
- 1 pt: Specific, testable constraint. "NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST" is testable. "Be thorough" is not.

**Scope Prevention [1 pt]**
- 0 pts: Agent can freely exceed the skill's intended scope
- 1 pt: Iron Law explicitly prevents the most common scope creep. Brainstorming skills gate out implementation. Investigation skills gate out premature fixes.

### Phase-Based Workflow [5 pts]

**Numbered phases [1 pt]**
- 0 pts: Instructions are a wall of text with no phases
- 1 pt: Phases are numbered and named (Phase 1: Context Gathering, etc.)

**Clear inputs [1 pt]**
- 0 pts: Phases start with vague instructions like "look at the code"
- 1 pt: Each phase specifies what to read, check, or receive. Commands are given explicitly.

**Clear outputs [1 pt]**
- 0 pts: Phases end without a defined deliverable
- 1 pt: Each phase produces something concrete (a list, a classification, a diff, a report)

**Standard pattern [1 pt]**
- 0 pts: Random ordering of activities
- 1 pt: Follows Context -> Analysis -> Action -> Verification -> Report (or justified deviation)

**Transition criteria [1 pt]**
- 0 pts: No boundary between phases
- 1 pt: Explicit criteria for when to move to the next phase

### Verification [4 pts]

**Phase exists [1 pt]**
- 0 pts: No verification phase
- 1 pt: Explicit verification phase or section

**Evidence required [1 pt]**
- 0 pts: "Make sure it works" with no proof mechanism
- 1 pt: Must produce evidence (test output, screenshot, diff, command result)

**Fresh evidence [1 pt]**
- 0 pts: Allows reuse of old evidence or assumptions
- 1 pt: Requires re-running checks after changes. "Code changed since then. Test again."

**Rationalization prevention [1 pt]**
- 0 pts: No guardrails against skipping verification
- 1 pt: Explicit rules: "Should work now" -> RUN IT. "I'm confident" -> Confidence is not evidence.

### Stop Conditions [4 pts]

**"Only stop for" list [2 pts]**
- 0 pts: No stop conditions defined
- 1 pt: Stop conditions exist but are vague or incomplete
- 2 pts: Specific, testable conditions covering safety (security uncertainty), blockers (merge conflicts), and errors (3-strike rule)

**"Never stop for" list [2 pts]**
- 0 pts: No "never stop for" list (agent pauses for every trivial decision)
- 1 pt: List exists but is sparse (covers 1-2 items)
- 2 pts: Covers the most common agent hesitations for this skill type. Ship skill: uncommitted changes, version bump choice, changelog content. QA skill: test framework choice, browser selection.

### Voice Directive [3 pts]

**Tone definition [1 pt]**
- 0 pts: No voice guidance
- 1 pt: Clear tone description ("direct, concrete, sharp, never corporate")

**Banned vocabulary [1 pt]**
- 0 pts: No banned words
- 1 pt: Explicit list of banned AI slop words (delve, crucial, robust, etc.)

**Writing rules [1 pt]**
- 0 pts: No formatting guidance
- 1 pt: Specific rules about em dashes, paragraph length, naming specifics, ending with actions

---

## Category 3: Development Infrastructure (20 points)

### Tests [7 pts]

**Test suite exists [2 pts]**
- 0 pts: No tests at all
- 1 pt: Tests exist but don't run reliably or are incomplete
- 2 pts: Full test suite that runs with a single command and passes

**Static validation (Tier 1) [2 pts]**
- 0 pts: No static checks
- 1 pt: Basic linting only
- 2 pts: Parses skill content, validates command references against registry, checks structural requirements. Fast (<2s) and free.

**Integration/E2E (Tier 2) [2 pts]**
- 0 pts: No integration or E2E tests
- 1 pt: Some integration tests but not covering primary workflows
- 2 pts: E2E tests that spawn real agent sessions and verify end-to-end behavior

**LLM-as-judge (Tier 3) [1 pt]**
- 0 pts: No quality scoring
- 1 pt: LLM-based evaluation that scores output quality (clarity, completeness, actionability)

### CI/CD [5 pts]

**Pipeline exists [2 pts]**
- 0 pts: No CI
- 1 pt: CI config exists but is broken or incomplete
- 2 pts: Working CI pipeline (GitHub Actions, CircleCI, etc.)

**Runs on push/PR [1 pt]**
- 0 pts: CI is manual-only
- 1 pt: CI triggers automatically on push or PR

**Doc staleness check [1 pt]**
- 0 pts: Generated docs can drift without detection
- 1 pt: CI runs doc generation in dry-run mode and fails if output differs from committed version

**Merge blocking [1 pt]**
- 0 pts: CI failures don't block merge
- 1 pt: Failed CI prevents merge to main branch

### Build Scripts [4 pts]

**Package manifest [1 pt]**
- 0 pts: No package.json or equivalent
- 1 pt: Manifest exists with scripts for build, test, and lint

**Template compilation [1 pt]**
- 0 pts: No template system (skills are hand-written) -- acceptable if <2 skills, 0 pts if 2+ skills
- 1 pt: Script compiles templates to SKILL.md files

**Setup script [1 pt]**
- 0 pts: Multi-step manual setup
- 1 pt: One-command `./setup` or equivalent that handles dependencies, builds, and configuration

**Compiled output [1 pt]**
- 0 pts: Raw source code runs at runtime with node_modules
- 1 pt: Build produces compiled/bundled output (Bun compile, webpack, esbuild, etc.)

### Package Management [4 pts]

**Lock file [1 pt]**
- 0 pts: No lock file committed
- 1 pt: Lock file present and committed

**Minimal dependencies [1 pt]**
- 0 pts: Kitchen-sink dependencies with many unused packages
- 1 pt: Each dependency is justified and necessary

**No unused deps [1 pt]**
- 0 pts: Multiple dependencies in manifest that aren't imported anywhere
- 1 pt: Every listed dependency is actually used

**Version constraints [1 pt]**
- 0 pts: All dependencies use `*` or `latest`
- 1 pt: Dependencies are pinned or use reasonable ranges (^, ~)

---

## Category 4: Code Quality Patterns (15 points)

### Error Handling [5 pts]

**Actionable errors [1 pt]**
- 0 pts: Error messages are generic ("Error occurred", "Failed")
- 1 pt: Every error includes what went wrong AND what to do about it

**Recovery suggestions [1 pt]**
- 0 pts: Errors just report failure
- 1 pt: Errors suggest the next command to run or the likely fix

**Graceful degradation [1 pt]**
- 0 pts: Any error crashes the process
- 1 pt: Non-fatal errors allow continued operation with fallback behavior

**3-strike escalation [1 pt]**
- 0 pts: No limit on retry attempts (agent spins forever)
- 1 pt: After N failures (typically 3), the system stops and escalates

**Agent-oriented messages [1 pt]**
- 0 pts: Errors are written for human debugging ("something went wrong")
- 1 pt: Errors are written so an AI agent can parse them and take corrective action

### Security [5 pts]

**No hardcoded secrets [1 pt]**
- 0 pts: Tokens, keys, or credentials in source code
- 1 pt: All secrets come from environment variables, config files, or secret managers

**Localhost binding [1 pt]**
- 0 pts: Daemon/server processes bind to 0.0.0.0 or public interfaces
- 1 pt: All inter-process servers bind to localhost only

**Auth for IPC [1 pt]**
- 0 pts: No authentication on inter-process communication
- 1 pt: Bearer token or equivalent authentication on all IPC channels

**No plaintext sensitive data [1 pt]**
- 0 pts: Cookies, sessions, or keys stored in plaintext files
- 1 pt: Sensitive data encrypted or stored in secure system facilities

**Security escalation [1 pt]**
- 0 pts: Agent makes security decisions autonomously
- 1 pt: Any security-uncertain decision escalates to the user with "STOP and escalate"

### Consistency [5 pts]

**Naming conventions [1 pt]**
- 0 pts: Mixed naming styles (camelCase and snake_case in the same file)
- 1 pt: Consistent naming throughout (pick one style and stick with it)

**Predictable structure [1 pt]**
- 0 pts: Files scattered randomly
- 1 pt: Directory structure follows a pattern. All skills in one place, all scripts in another.

**Linter enforcement [1 pt]**
- 0 pts: No linter configuration
- 1 pt: Linter config present (ESLint, Biome, Prettier, etc.) and enforced in CI

**Same patterns for same problems [1 pt]**
- 0 pts: Ad-hoc solutions. Error handling done differently in each file.
- 1 pt: Consistent patterns. All HTTP calls handle errors the same way. All skills use the same output format.

**Comments explain WHY [1 pt]**
- 0 pts: Comments restate the code ("increment counter") or are absent
- 1 pt: Comments explain decisions ("Use setTimeout instead of setInterval because intervals can stack during long operations")

---

## Category 5: Workflow Coverage (15 points)

### Planning Phase [3 pts]

**Brainstorming skill [1 pt]**
- 0 pts: No ideation or brainstorming capability
- 1 pt: A skill for validating ideas, exploring problem spaces, or design thinking (office-hours equivalent)

**Strategy review [1 pt]**
- 0 pts: No strategic scope review
- 1 pt: A skill for challenging premises, expanding/reducing scope, and validating strategy (CEO review equivalent)

**Architecture review [1 pt]**
- 0 pts: No architectural review
- 1 pt: A skill for reviewing technical decisions, data flow, edge cases, and test coverage (eng review equivalent)

### Building Phase [3 pts]

**Implementation guidance [1 pt]**
- 0 pts: No scaffolding or implementation help
- 1 pt: Skills that help with code generation, scaffolding, or implementation patterns

**Design review [1 pt]**
- 0 pts: No design capability
- 1 pt: Visual design, design system, or UX review skill

**Debugging [1 pt]**
- 0 pts: No debugging methodology
- 1 pt: Structured debugging skill with root cause investigation (investigate equivalent)

### Testing Phase [3 pts]

**QA testing [1 pt]**
- 0 pts: No QA capability
- 1 pt: Test + fix loop skill that finds bugs and fixes them with verification

**Code review [1 pt]**
- 0 pts: No pre-landing review
- 1 pt: Diff analysis skill for catching issues before merge

**Report-only testing [1 pt]**
- 0 pts: No way to test without fixing (report-only mode missing)
- 1 pt: Option to produce a test report without making changes

### Shipping Phase [3 pts]

**Ship skill [1 pt]**
- 0 pts: No automated shipping workflow
- 1 pt: Skill handles version bump, changelog, commit, push, and PR creation

**Doc update [1 pt]**
- 0 pts: Documentation gets stale after shipping
- 1 pt: Post-ship skill updates README, ARCHITECTURE, CHANGELOG to match what shipped

**Retrospective [1 pt]**
- 0 pts: No retrospective capability
- 1 pt: Skill analyzes what shipped, commit patterns, and areas for improvement

### Deployment and Monitoring [3 pts]

**Deploy skill [1 pt]**
- 0 pts: No deployment automation
- 1 pt: Merge, deploy, and production verification in one workflow

**Canary monitoring [1 pt]**
- 0 pts: No post-deploy monitoring
- 1 pt: Post-deploy health checks, error monitoring, or canary analysis

**Workflow graph [1 pt]**
- 0 pts: Skills are isolated islands with no cross-references
- 1 pt: Each skill suggests the next skill in the pipeline. The full lifecycle is connected.

---

## Scoring Quick Reference

| Category | Full Points Example | Zero Points Example |
|----------|-------------------|-------------------|
| ETHOS.md | Specific philosophy that shapes every skill | Generic "we value quality" mission statement |
| CLAUDE.md | Copy-pasteable commands, directory tree, conventions | Empty or "TODO" |
| Frontmatter | 4-line description with triggers, minimal allowed-tools, correct tier | Single-line description, no tool whitelist |
| Iron Law | "NO FIXES WITHOUT ROOT CAUSE" | "Be careful" |
| Phases | 5 numbered phases with inputs, outputs, transitions | Wall of text instructions |
| Verification | Fresh evidence required, rationalization prevention | "Make sure it works" |
| Stop conditions | Both "only stop for" and "never stop for" with 3+ items each | Neither list present |
| Tests | 3-tier (static + E2E + LLM judge), single-command | No tests |
| Voice | Tone + banned words + writing rules + final test | No voice guidance |
| Workflow | Full pipeline from brainstorm to monitor, skills linked | Isolated skills with no awareness of each other |
