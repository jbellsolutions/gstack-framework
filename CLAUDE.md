# gstack-framework development

## What this is

A Claude Code skill pack that transforms any repo to G-Stack quality.
8 skills, 9 templates, 3 checklists, full reference material.

## Structure

```
gstack-framework/
|-- skills/              # 8 active skills (the tools)
|   |-- orchestrator/    # /gstack-framework - main entry point
|   |-- analyze/         # /gf-analyze - scan repo
|   |-- audit/           # /gf-audit - score 0-100
|   |-- find-skills/     # /gf-find-skills - identify skill gaps
|   |-- find-agents/     # /gf-find-agents - identify agent opportunities
|   |-- create-skills/   # /gf-create-skills - generate SKILL.md files
|   |-- apply/           # /gf-apply - apply full template
|   |-- verify/          # /gf-verify - verify and score
|-- templates/           # Raw templates with {{PLACEHOLDER}} syntax
|-- checklists/          # Scoring rubrics
|-- reference/           # Analysis + gstack clone
```

## Key conventions

- Skills are standalone markdown files. No build step.
- Templates use `{{PLACEHOLDER}}` syntax. Skills read and fill them.
- Checklists are scored rubrics. Skills use them to validate output.
- The orchestrator chains all skills: analyze -> audit -> find-skills -> find-agents -> create-skills -> apply -> verify.
- Non-destructive: never overwrite without asking.
- Every generated SKILL.md must pass checklists/skill-quality.md before writing.

## Installation

```bash
git clone <repo> ~/.claude/skills/gstack-framework
cd ~/.claude/skills/gstack-framework && ./setup
```

## Adding a new skill to the framework

1. Create `skills/{name}/SKILL.md`
2. Follow the SKILL.md template in `templates/SKILL.md.tmpl`
3. Ensure it passes `checklists/skill-quality.md`
4. Update the orchestrator to include it in the pipeline

## Quality checklist (for this repo)

- Every skill has YAML frontmatter with name, description, allowed-tools
- Every skill has phase-based workflow
- Every skill has verification step
- Every skill has voice directive
- Every skill has completion status protocol
- Templates have clear {{PLACEHOLDER}} markers
- Checklists have point values that sum correctly
- README is current with all skills listed

---

## Session startup checklist

At the start of every session, do these in order:

1. Read this CLAUDE.md fully
2. Read `checklists/skill-quality.md` and `checklists/repo-quality.md`
3. Check `claude-progress.txt` for where the last session left off
4. Check `.claude/healing/history.json` for recent errors to avoid repeating
5. Check `.claude/learning/observations.json` for accumulated context
6. Summarize: what was done, what is next, any blockers

## Session end instructions

Before ending any session:

1. Update `claude-progress.txt` with what was accomplished and what comes next
2. Log any new error patterns to `.claude/healing/history.json`
3. Log any new observations to `.claude/learning/observations.json`
4. If an insight was confirmed 3+ times, promote it to `.claude/learning/insights.json`
5. Never leave work in a half-applied state. Finish the current phase or revert.

## Compaction rules

When context runs low and compaction is needed:

- Always preserve: current task, file paths being edited, test results, error messages
- Always preserve: the contents of this CLAUDE.md (re-read after compaction)
- Always preserve: in-progress checklist scores and audit state
- Safe to drop: file contents already read and processed, verbose command output, intermediate search results
- After compaction: re-read `claude-progress.txt` to restore session state

## Search strategy

When looking for code, patterns, or files in this repo:

1. Start with `skills/` for active skill implementations
2. Check `templates/` for raw template files with `{{PLACEHOLDER}}` syntax
3. Check `checklists/` for scoring rubrics and quality criteria
4. Check `reference/` for analysis docs and the original gstack clone
5. Use Glob for file discovery, Grep for content search
6. Never assume a file exists. Verify first.

## Thinking guidelines

Before modifying any framework skill, template, or checklist:

- Think through backward compatibility. Repos already using gstack depend on the current skill names, template placeholders, and checklist point values.
- Changes to `skill-quality.md` or `repo-quality.md` affect every repo that runs `/gf-audit` or `/gf-verify`. Do not change scoring without updating all references.
- Changes to template `{{PLACEHOLDER}}` names break the `apply` skill. If a placeholder must change, update `skills/apply/SKILL.md` in the same commit.
- Adding new skills is safe. Modifying existing skill interfaces requires a deprecation path.
- When in doubt, add alongside rather than replace.
