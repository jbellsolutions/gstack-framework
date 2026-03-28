# GEMINI.md -- Gemini AI context for gstack-framework

## What this repo is

A Claude Code skill pack that transforms any repo to G-Stack quality. 8 skills, 9 templates, 3 checklists, full reference material.

## Repo structure

```
gstack-framework/
|-- skills/              # 8 active skills (SKILL.md files)
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

- Skills are standalone markdown files with YAML frontmatter. No build step.
- Templates use `{{PLACEHOLDER}}` syntax. Skills read and fill them.
- Checklists are scored rubrics. Skills use them to validate output.
- The orchestrator chains all skills: analyze -> audit -> find-skills -> find-agents -> create-skills -> apply -> verify.
- Non-destructive: never overwrite without asking.
- Every generated SKILL.md must pass checklists/skill-quality.md before writing.

## Important constraints

- Do not modify existing skill interfaces without a deprecation path.
- Do not change checklist point values without updating all references.
- Template placeholder names are part of the public API. Changing them breaks the apply skill.
- Adding new skills alongside existing ones is always safe.

## Quality standards

Every SKILL.md must have:
- YAML frontmatter (name, description with triggers, allowed-tools)
- Phase-based workflow (Context -> Analysis -> Action -> Verification -> Report)
- Verification step with evidence requirement
- Stop conditions (both "stop for" and "never stop for")
- Voice directive with banned AI slop vocabulary
- Completion Status Protocol (DONE, BLOCKED, NEEDS_CONTEXT)
