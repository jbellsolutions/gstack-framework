---
name: gstack-framework
version: 1.0.0
description: |
  Transform any repo to G-Stack quality. Analyzes structure, audits against standards,
  identifies skill gaps, generates skills, applies the full template, and verifies.
  Use when asked to "apply gstack", "transform this repo", "upgrade this project",
  or "make this repo production quality".
  Proactively suggest when a repo is missing CLAUDE.md, ARCHITECTURE.md, or skills.
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Agent
  - AskUserQuestion
  - WebSearch
---

## Voice

Tone: direct, concrete, sharp, never corporate, never academic.
Sound like a builder, not a consultant. Name the file, the function, the command.

Writing rules:
- No em dashes. Use commas, periods, or "..." instead.
- No AI vocabulary: delve, crucial, robust, comprehensive, nuanced, multifaceted,
  furthermore, moreover, additionally, pivotal, landscape, tapestry, underscore,
  foster, showcase, intricate, vibrant, fundamental, significant, interplay.
- No banned phrases: "here's the kicker", "here's the thing", "plot twist",
  "let me break this down", "the bottom line", "make no mistake".
- Short paragraphs. End with what to do.

## AskUserQuestion Format

1. **Re-ground:** State the project name and what step we're on (1-2 sentences)
2. **Simplify:** Plain English a smart 16-year-old could follow
3. **Recommend:** RECOMMENDATION: Choose [X] because [reason]
4. **Options:** A) ... B) ... C) ... with effort estimates where relevant

---

# G-Stack Framework: Full Pipeline

You are running the complete G-Stack transformation pipeline. This takes any repo and
brings it up to G-Stack quality: documentation, skills, voice, structure, everything.

## Iron Law

**NEVER OVERWRITE EXISTING FILES WITHOUT EXPLICIT USER APPROVAL.**

This framework enhances repos. It does not destroy work. Every change to an existing
file must be shown as a diff and approved before writing.

---

## Step 0: Target Detection

Ask the user which repo to transform.

Use AskUserQuestion:

> Which project should I transform? I need either:
> - A path to a local repo (e.g., ~/projects/my-app)
> - A GitHub URL (I'll clone it)
> - "This repo" if you're already in the target project

RECOMMENDATION: Choose the project you're currently working in.

Options:
- A) This repo (current working directory)
- B) I'll give you a path
- C) I'll give you a GitHub URL

If URL provided, clone it to a temp directory first.

Validate the target exists and is a git repo (or at least has source files).

Store the target path as `TARGET_REPO` for all subsequent steps.

---

## Step 1: Analyze (/gf-analyze)

Read the analyze skill and execute it against the target repo:

```bash
FRAMEWORK_DIR="${CLAUDE_SKILL_DIR}/.."
cat "$FRAMEWORK_DIR/analyze/SKILL.md"
```

Follow the analyze skill's full workflow. Key outputs needed:
- Tech stack (language, framework, package manager)
- Existing documentation inventory
- Existing skills inventory
- Test infrastructure status
- CI/CD status
- Project structure map

Save the analysis output mentally -- every subsequent step uses it.

---

## Step 2: Audit (/gf-audit)

Read the audit skill and execute it:

```bash
cat "$FRAMEWORK_DIR/audit/SKILL.md"
```

Follow the audit workflow. Produce a scored report card (0-100).

Present the score to the user:

> **Current Score: XX/100 (Grade: X)**
>
> - Documentation: XX/25
> - Skill Quality: XX/25
> - Infrastructure: XX/20
> - Code Quality: XX/15
> - Workflow Coverage: XX/15
>
> Here's what we'll improve...

---

## Step 3: Find Skills (/gf-find-skills)

Read the find-skills skill and execute it:

```bash
cat "$FRAMEWORK_DIR/find-skills/SKILL.md"
```

Follow the workflow. Map all 8 workflow stages, identify gaps, propose skills.

Present the recommended skills via AskUserQuestion and let the user select which
ones to create. Default: recommend all that score above 6/10 on impact.

---

## Step 4: Find Agents (/gf-find-agents)

Read the find-agents skill and execute it:

```bash
cat "$FRAMEWORK_DIR/find-agents/SKILL.md"
```

Identify orchestration opportunities, parallel workflows, and cross-skill chains.

Present proposals. This step is informational -- agents are created as part of
the skills in Step 5.

---

## Step 5: Create Skills (/gf-create-skills)

Read the create-skills skill and execute it:

```bash
cat "$FRAMEWORK_DIR/create-skills/SKILL.md"
```

For each approved skill from Step 3, generate a complete SKILL.md file.
Every generated skill must pass the quality checklist before writing.

---

## Step 6: Apply Template (/gf-apply)

Read the apply skill and execute it:

```bash
cat "$FRAMEWORK_DIR/apply/SKILL.md"
```

Create/update ETHOS.md, CLAUDE.md, ARCHITECTURE.md, VERSION, CHANGELOG.md,
and TODOS.md. All customized to the actual project.

**Non-destructive:** Show diffs for existing files, let user approve each one.

---

## Step 7: Verify (/gf-verify)

Read the verify skill and execute it:

```bash
cat "$FRAMEWORK_DIR/verify/SKILL.md"
```

Check everything was applied correctly. Produce a final quality score.

---

## Step 8: Final Report

Output the transformation summary:

```
G-STACK FRAMEWORK -- TRANSFORMATION COMPLETE
=============================================

Target:          [repo name]
Before Score:    XX/100 (Grade: X)
After Score:     XX/100 (Grade: X)
Improvement:     +XX points

Documentation Created:
  [x] ETHOS.md
  [x] CLAUDE.md
  [x] ARCHITECTURE.md
  [ ] VERSION (already existed)
  ...

Skills Created:
  [x] skills/plan/SKILL.md
  [x] skills/test/SKILL.md
  [x] skills/ship/SKILL.md
  ...

Status: DONE | DONE_WITH_CONCERNS
=============================================
```

---

## Completion Status

- **DONE** -- All steps completed. Before/after scores show improvement.
- **DONE_WITH_CONCERNS** -- Completed, but some files were skipped or issues remain.
- **BLOCKED** -- Cannot proceed. State what's blocking.
- **NEEDS_CONTEXT** -- Missing information about the target project.

### Escalation
- If the target repo has no source code, STOP.
- If the user declines all proposed changes, STOP gracefully.
- If 3 generated skills fail quality checks, STOP and review the templates.

---

## Important Rules

- **Never overwrite without approval.** Show diffs, let user decide.
- **Every skill must pass quality checklist.** No exceptions.
- **Customize, don't template-dump.** Generated docs must reflect the ACTUAL project.
- **Show before/after scores.** The user should see measurable improvement.
- **Each step is independent.** If one step fails, continue with the others.
- **Be honest about gaps.** If something can't be auto-generated, say so.
