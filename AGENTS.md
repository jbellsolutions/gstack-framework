# AGENTS.md -- Agent coordination for gstack-framework

## Overview

This file defines how AI agents should interact with the gstack-framework repo. It covers roles, boundaries, and coordination patterns.

## Agent roles

### Primary: Framework Developer
- Owns: skills/, templates/, checklists/
- Responsibilities: create new skills, update templates, maintain checklists
- Constraints: never modify existing skill interfaces without deprecation path, never change checklist point totals without updating all references

### Security Reviewer (.claude/agents/security-reviewer.md)
- Focus: skill injection safety, template integrity, checklist tampering
- Trigger: any PR that modifies skills/ or templates/
- Authority: can block merge on security grounds

### Code Reviewer (.claude/agents/code-reviewer.md)
- Focus: skill quality checklist compliance
- Trigger: any PR that adds or modifies SKILL.md files
- Authority: can request changes, cannot merge

## Coordination rules

1. One agent works on one skill at a time. No parallel edits to the same SKILL.md.
2. The orchestrator skill chains all other skills. Do not bypass the chain unless running a single skill explicitly.
3. Template changes require corresponding skill updates in the same commit.
4. Checklist changes require re-audit of all existing skills.

## File ownership

| Path | Owner | Review required |
|------|-------|----------------|
| skills/ | Framework Developer | Code Reviewer |
| templates/ | Framework Developer | Code Reviewer |
| checklists/ | Framework Developer | Security Reviewer + Code Reviewer |
| reference/ | Read-only | None (reference material) |
| CLAUDE.md | Framework Developer | None |
| setup | Framework Developer | Security Reviewer |

## Communication protocol

Agents communicate through:
- `claude-progress.txt` -- session state and handoff notes
- `.claude/learning/observations.json` -- accumulated context across sessions
- `.claude/healing/history.json` -- error patterns to avoid
- Git commit messages -- what changed and why
