# Repo Quality Audit Checklist

**Scoring:** 0-100 points across 5 categories. Each item has a point value in brackets.
**Pass threshold:** 70/100 (ship-ready), 85/100 (production-grade), 95/100 (reference-quality)

---

## Category 1: Documentation (25 points)

Foundation files that define the repo's identity, philosophy, and developer experience.

### ETHOS.md [5 pts]
- [ ] [2] ETHOS.md exists at repo root
- [ ] [1] Contains a Completeness Principle (lakes vs oceans, marginal cost reasoning)
- [ ] [1] Contains a Search Before Building directive with layered knowledge model
- [ ] [1] Contains a Build for Yourself or equivalent specificity principle

### CLAUDE.md [5 pts]
- [ ] [1] CLAUDE.md exists at repo root
- [ ] [1] Lists all dev commands (install, test, build, lint) with exact syntax
- [ ] [1] Documents project structure with directory tree
- [ ] [1] Lists key conventions (naming, patterns, file organization)
- [ ] [1] Includes AI effort compression table or equivalent guidance

### ARCHITECTURE.md [5 pts]
- [ ] [1] ARCHITECTURE.md exists at repo root
- [ ] [1] Explains the core idea in one paragraph (problem + key insight)
- [ ] [1] Justifies technology choices with concrete reasons (not just popularity)
- [ ] [1] Documents what is intentionally NOT included, with reasons
- [ ] [1] Describes error philosophy (actionable messages for agents, not humans)

### README.md [4 pts]
- [ ] [1] README.md exists with project description
- [ ] [1] Contains installation instructions (one-command preferred)
- [ ] [1] Contains usage examples
- [ ] [1] Contains license and contribution pointers

### CHANGELOG.md + VERSION [3 pts]
- [ ] [1] CHANGELOG.md exists with user-facing release notes
- [ ] [1] VERSION file exists with semver (3-digit minimum, 4-digit preferred)
- [ ] [1] CHANGELOG entries match VERSION bumps chronologically

### TODOS.md [3 pts]
- [ ] [1] TODOS.md exists with prioritized work items
- [ ] [1] Items are categorized by priority level (P0-P4 or equivalent)
- [ ] [1] Items are actionable (not vague wishes, but concrete next steps)

---

## Category 2: Skill Quality (25 points)

Structure and craftsmanship of individual SKILL.md files. Score the average across all skills.

### Frontmatter [6 pts]
- [ ] [1] Every SKILL.md has YAML frontmatter with `name` field
- [ ] [1] Every SKILL.md has a multi-line `description` with trigger phrases ("Use when asked to...")
- [ ] [1] Descriptions include proactive trigger conditions ("Proactively suggest when...")
- [ ] [1] Descriptions include disambiguation ("For X, use /other instead")
- [ ] [1] `allowed-tools` whitelist is present and minimal (no unnecessary tools)
- [ ] [1] `preamble-tier` is set (1-4) and appropriate for skill complexity

### Iron Law / Hard Gate [3 pts]
- [ ] [1] Every complex skill has an Iron Law or Hard Gate section
- [ ] [1] The constraint is specific and actionable (not vague)
- [ ] [1] The constraint prevents scope creep or dangerous behavior

### Phase-Based Workflow [5 pts]
- [ ] [1] Skills use numbered phases (Phase 1, Phase 2, etc.)
- [ ] [1] Each phase has clear inputs (what it reads/checks)
- [ ] [1] Each phase has clear outputs (what it produces/changes)
- [ ] [1] Phases follow the pattern: Context -> Analysis -> Action -> Verification -> Report
- [ ] [1] Phase transitions have explicit criteria (when to move to next phase)

### Verification [4 pts]
- [ ] [1] Verification phase exists with evidence requirement
- [ ] [1] Rationalization prevention rules are present ("Should work now" -> RUN IT)
- [ ] [1] Verification requires fresh evidence (not cached or assumed)
- [ ] [1] Failed verification triggers re-work, not skip

### Stop Conditions [4 pts]
- [ ] [2] "Only stop for" list is present with specific conditions
- [ ] [2] "Never stop for" list is present with trivial items agents waste time on

### Voice Directive [3 pts]
- [ ] [1] Voice section exists with tone definition
- [ ] [1] Banned AI vocabulary list is present (delve, crucial, robust, etc.)
- [ ] [1] Writing rules are specific (no em dashes, short paragraphs, name the file)

---

## Category 3: Development Infrastructure (20 points)

Build tooling, testing, and automation that keeps quality consistent.

### Tests [7 pts]
- [ ] [2] Test suite exists and runs with a single command
- [ ] [2] Static validation tests exist (Tier 1: free, fast, parse-based)
- [ ] [2] Integration or E2E tests exist (Tier 2: real sessions or workflows)
- [ ] [1] LLM-as-judge or quality scoring tests exist (Tier 3: output evaluation)

### CI/CD [5 pts]
- [ ] [2] CI pipeline exists (GitHub Actions, etc.)
- [ ] [1] CI runs tests on every push or PR
- [ ] [1] CI validates generated docs are not stale (dry-run + diff check)
- [ ] [1] CI blocks merge on test failure

### Build Scripts [4 pts]
- [ ] [1] `package.json` (or equivalent) exists with build/test/lint scripts
- [ ] [1] Template compilation script exists (if 2+ skills)
- [ ] [1] One-command installer/setup script exists
- [ ] [1] Build produces compiled/bundled output (no raw source at runtime)

### Package Management [4 pts]
- [ ] [1] Lock file is committed (package-lock.json, bun.lockb, etc.)
- [ ] [1] Dependencies are minimal and justified
- [ ] [1] No unused dependencies in manifest
- [ ] [1] Versions are pinned or range-constrained

---

## Category 4: Code Quality Patterns (15 points)

Consistency, error handling, and security across the codebase.

### Error Handling [5 pts]
- [ ] [1] Errors are actionable ("Element not found. Run `snapshot -i` to see available elements.")
- [ ] [1] Errors include recovery suggestions, not just failure messages
- [ ] [1] Graceful degradation with fallbacks where appropriate
- [ ] [1] 3-strike escalation rule for repeated failures
- [ ] [1] Error messages are written for AI agents, not just humans

### Security [5 pts]
- [ ] [1] No hardcoded secrets, tokens, or credentials
- [ ] [1] Localhost-only binding for any daemon/server processes
- [ ] [1] Bearer token or equivalent auth for inter-process communication
- [ ] [1] No plaintext storage of sensitive data (cookies, sessions, keys)
- [ ] [1] Escalation to user on security-uncertain decisions

### Consistency [5 pts]
- [ ] [1] Naming conventions are consistent across all files
- [ ] [1] File structure follows a predictable pattern
- [ ] [1] Code style is uniform (linter config present and enforced)
- [ ] [1] Same patterns used for similar problems (no ad-hoc solutions)
- [ ] [1] Comments explain WHY, not WHAT

---

## Category 5: Workflow Coverage (15 points)

Does the skill set cover the full software development lifecycle?

### Planning Phase [3 pts]
- [ ] [1] Brainstorming/ideation skill exists (office-hours equivalent)
- [ ] [1] Strategy/scope review skill exists (CEO review equivalent)
- [ ] [1] Architecture/eng review skill exists

### Building Phase [3 pts]
- [ ] [1] Implementation guidance or scaffolding skills exist
- [ ] [1] Design review or design system skill exists
- [ ] [1] Debugging/investigation skill exists

### Testing Phase [3 pts]
- [ ] [1] QA testing skill exists (test + fix loop)
- [ ] [1] Code review skill exists (pre-landing diff analysis)
- [ ] [1] Report-only testing option exists (audit without fixes)

### Shipping Phase [3 pts]
- [ ] [1] Ship/PR creation skill exists (version bump, changelog, push)
- [ ] [1] Documentation update skill exists (post-ship doc sync)
- [ ] [1] Retrospective skill exists (what shipped, what to improve)

### Deployment and Monitoring [3 pts]
- [ ] [1] Deploy/land skill exists (merge + deploy + verify)
- [ ] [1] Canary/monitoring skill exists (post-deploy health checks)
- [ ] [1] Skills suggest adjacent skills in the pipeline (workflow graph)

---

## Scoring Summary

| Category | Max Points | Score |
|----------|-----------|-------|
| Documentation | 25 | __/25 |
| Skill Quality | 25 | __/25 |
| Development Infrastructure | 20 | __/20 |
| Code Quality Patterns | 15 | __/15 |
| Workflow Coverage | 15 | __/15 |
| **Total** | **100** | **__/100** |

### Rating Scale

| Score | Rating | Meaning |
|-------|--------|---------|
| 95-100 | Reference Quality | Could be used as the canonical example for others |
| 85-94 | Production Grade | Solid, trustworthy, ready for teams to depend on |
| 70-84 | Ship Ready | Functional with known gaps, acceptable for v1 |
| 50-69 | Draft Quality | Missing significant pieces, needs iteration |
| 0-49 | Prototype | Proof of concept only, not ready for use |
