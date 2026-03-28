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
