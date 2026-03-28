# G-Stack Framework

Transform any repo to G-Stack quality. One install, fully automated.

Point this framework at any project and it will analyze, audit, create skills, apply templates, and verify -- bringing your repo up to the standard of Garry Tan's gstack (the most sophisticated AI engineering workflow in production).

## What It Does

```
Your Repo (before)              Your Repo (after)
------------------              -----------------
README.md                       README.md (improved)
src/                            src/
package.json           -->      package.json
                                ETHOS.md (builder philosophy)
                                CLAUDE.md (dev guide + commands)
                                ARCHITECTURE.md (technical decisions)
                                VERSION (4-digit semver)
                                CHANGELOG.md (user-facing notes)
                                TODOS.md (prioritized work)
                                skills/
                                  plan/SKILL.md
                                  build/SKILL.md
                                  test/SKILL.md
                                  ship/SKILL.md
                                  ... (custom to your project)
```

## Quick Start

```bash
# 1. Clone into your Claude Code skills
git clone https://github.com/YOUR_USER/gstack-framework.git ~/.claude/skills/gstack-framework
cd ~/.claude/skills/gstack-framework && ./setup

# 2. Open any project in Claude Code and run:
/gstack-framework
```

That's it. The orchestrator walks you through everything:

1. **Analyze** -- Scans your repo structure, tech stack, docs, tests, CI/CD
2. **Audit** -- Scores it 0-100 against G-Stack quality standards
3. **Find Skills** -- Identifies where skills are needed across your workflow
4. **Find Agents** -- Identifies multi-agent and chaining opportunities
5. **Create Skills** -- Generates high-quality SKILL.md files for each gap
6. **Apply Template** -- Adds ETHOS.md, CLAUDE.md, ARCHITECTURE.md, etc.
7. **Verify** -- Checks everything was applied correctly, produces final score

## Individual Skills

Run any step independently:

| Skill | What it does |
|-------|-------------|
| `/gf-analyze` | Scan repo structure, tech stack, existing docs and skills |
| `/gf-audit` | Score repo 0-100 against G-Stack standards |
| `/gf-find-skills` | Identify where skills should exist across the dev lifecycle |
| `/gf-find-agents` | Find multi-agent and cross-skill chaining opportunities |
| `/gf-create-skills` | Generate SKILL.md files for approved skills |
| `/gf-apply` | Apply full G-Stack template (ETHOS, CLAUDE.md, ARCHITECTURE, etc.) |
| `/gf-verify` | Verify all components applied correctly, produce final score |

## What Gets Created

### Documentation (customized to YOUR project)
- **ETHOS.md** -- Your builder philosophy (completeness principle, search-before-building)
- **CLAUDE.md** -- Dev commands, project structure, conventions (from actual package.json/Makefile)
- **ARCHITECTURE.md** -- Technical decisions and WHY (from actual tech stack)
- **VERSION** -- 4-digit semver (MAJOR.MINOR.PATCH.MICRO)
- **CHANGELOG.md** -- User-facing release notes format
- **TODOS.md** -- Prioritized work items (P0-P4)

### Skills (customized to YOUR workflow)
Each generated skill follows G-Stack patterns:
- YAML frontmatter with trigger phrases
- Phase-based workflow (Context -> Analysis -> Action -> Verification -> Report)
- Explicit stop conditions AND "never stop for" lists
- Voice directive (bans AI slop vocabulary)
- AskUserQuestion format (re-ground, simplify, recommend, options)
- Completion Status Protocol (DONE, BLOCKED, NEEDS_CONTEXT)
- Evidence-based verification ("confidence is not evidence")

## What's Inside

```
gstack-framework/
|-- skills/              # The active tools (8 skills)
|-- templates/           # Raw templates with {{PLACEHOLDER}} syntax
|-- checklists/          # Scoring rubrics and quality criteria
|-- reference/           # Analysis docs + cloned gstack repo
|   |-- GSTACK-DEEP-ANALYSIS.md
|   |-- GSTACK-TEMPLATE-FRAMEWORK.md
|   |-- patterns.md
|   |-- anti-patterns.md
|   |-- gstack/          # The original G-Stack repo (reference)
|-- setup                # One-command installer
|-- CLAUDE.md            # Dev guide for this framework
```

## Quality Standards

Every generated skill must pass the skill quality checklist:
- Proper YAML frontmatter (name, description with triggers, allowed-tools)
- Iron Law / Hard Gate (the one constraint that cannot be violated)
- Phase-based workflow with numbered phases
- Verification step requiring evidence
- Stop conditions (both "stop for" and "never stop for")
- Voice directive with banned vocabulary
- Completion Status Protocol with escalation rules
- No AI slop (delve, crucial, robust, comprehensive, nuanced...)

## Non-Destructive

The framework never overwrites existing files without asking. For every file that already exists, it shows a diff and lets you approve or skip.

## Based On

This framework extracts and templatizes the patterns from [Garry Tan's gstack](https://github.com/garrytan/gstack) -- 28 production skills covering the entire AI engineering workflow. See `reference/GSTACK-DEEP-ANALYSIS.md` for the complete breakdown.

## License

MIT
