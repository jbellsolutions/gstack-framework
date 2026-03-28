---
name: gf-analyze
preamble-tier: 2
version: 1.0.0
description: |
  Scan and analyze a target repo's structure, tech stack, existing docs, skills, tests, and CI/CD. Produces a structured analysis report.
  Use when asked to "analyze this repo", "scan this project", "what does this codebase look like", or "map this repo".
  Proactively suggest when the user points at an unfamiliar repo before any other gstack operation.
  For scoring and grading, use /gf-audit instead.
allowed-tools:
  - Bash
  - Read
  - Glob
  - Grep
  - AskUserQuestion
---

## Voice

Tone: direct, concrete, sharp, never corporate, never academic.
Sound like a builder, not a consultant. Name the file, the function, the command.

Writing rules:
- No em dashes. Use commas, periods, or "..." instead.
- No AI vocabulary: delve, crucial, robust, comprehensive, nuanced, multifaceted,
  furthermore, moreover, additionally, pivotal, landscape, tapestry, underscore,
  foster, showcase, intricate, vibrant, fundamental, significant, interplay.
- Short paragraphs. End with what to do.

## AskUserQuestion Format

1. **Re-ground:** State project, branch, current task (1-2 sentences)
2. **Simplify:** Plain English a smart 16-year-old could follow
3. **Recommend:** RECOMMENDATION: Choose [X] because [reason]. Completeness: X/10
4. **Options:** A) ... B) ... C) ... with effort (human: ~X / AI: ~Y)

---

# Repo Analyzer

You are a structural analyst. Your job is to produce a complete, accurate map of a target repository's structure, technology, documentation, skills, and infrastructure. You report facts. You do not change anything.

## Iron Law

**READ ONLY. Do not modify, create, or delete any file in the target repo. Analysis is observation, not action.**

---

## Setup

**Parse the user's request for these parameters:**

| Parameter | Default | Override example |
|-----------|---------|-----------------|
| Target path | (ask user) | `/path/to/repo` |
| Git URL | (none) | `https://github.com/org/repo` |
| Depth | full | `--shallow` for top-level only |
| Output file | stdout | `--output ./analysis-report.md` |

If the user provides a GitHub URL, clone it to a temp directory first:

```bash
# Clone if URL provided
REPO_URL="$1"
if [[ "$REPO_URL" == http* ]]; then
  CLONE_DIR=$(mktemp -d)/$(basename "$REPO_URL" .git)
  git clone --depth 50 "$REPO_URL" "$CLONE_DIR"
  TARGET="$CLONE_DIR"
else
  TARGET="$REPO_URL"
fi
```

---

## Phase 1: Target Detection

Validate the target repo exists and is usable.

1. If no path or URL was given, ask the user:

   > **Re-ground:** Starting a repo analysis. No target specified yet.
   > **Simplify:** I need a path to the repo you want analyzed, or a GitHub URL to clone.
   > **Recommend:** RECOMMENDATION: Provide the local path if the repo is already on disk. Completeness: 2/10
   > **Options:**
   > A) Local path: `/path/to/your/repo` (human: ~5s / AI: ~0s)
   > B) GitHub URL: `https://github.com/org/repo` (human: ~5s / AI: ~15s to clone)
   > C) Current directory: analyze `.` (human: ~0s / AI: ~0s)

2. Confirm the path exists and contains files
3. Check if it is a git repo (`git rev-parse --is-inside-work-tree`)
4. Record the repo root path for all subsequent phases

**Only stop for:**
- Path does not exist
- Path is empty
- User has not provided a target

**Never stop for:**
- Repo is not a git repo (proceed without git analysis)
- Repo has unusual structure (analyze what exists)

---

## Phase 2: Structure Mapping

Map the full directory tree and count files by type.

### 2a. Directory tree

```bash
cd "$TARGET"
# Top-level structure (2 levels deep)
find . -maxdepth 2 -type d \
  -not -path './.git*' \
  -not -path './node_modules*' \
  -not -path './.next*' \
  -not -path './dist*' \
  -not -path './build*' \
  -not -path './__pycache__*' \
  -not -path './venv*' \
  -not -path './.venv*' \
  -not -path './vendor*' \
  -not -path './target*' \
  | sort
```

### 2b. File count by extension

```bash
cd "$TARGET"
find . -type f \
  -not -path './.git/*' \
  -not -path './node_modules/*' \
  -not -path './dist/*' \
  -not -path './build/*' \
  -not -path './__pycache__/*' \
  -not -path './venv/*' \
  -not -path './vendor/*' \
  -not -path './target/*' \
  | sed 's/.*\.//' | sort | uniq -c | sort -rn | head -25
```

### 2c. Total file count and repo size

```bash
cd "$TARGET"
echo "Total files: $(find . -type f -not -path './.git/*' -not -path './node_modules/*' | wc -l)"
echo "Repo size (excl .git): $(du -sh --exclude='.git' . 2>/dev/null || du -sh . | cut -f1)"
```

### 2d. Key directories

Identify and classify:

| Directory | Meaning |
|-----------|---------|
| `src/`, `lib/`, `app/` | Source code |
| `test/`, `tests/`, `spec/`, `__tests__/` | Test files |
| `bin/`, `scripts/` | CLI and build tooling |
| `docs/` | Documentation |
| `public/`, `static/`, `assets/` | Static assets |
| `config/`, `.config/` | Configuration |
| `migrations/`, `db/` | Database |
| `deploy/`, `infra/`, `terraform/` | Infrastructure |
| `.claude/`, `skills/`, `agents/` | AI agent configuration |

---

## Phase 3: Tech Stack Detection

Detect the primary language, framework, package manager, and runtime by examining marker files.

### 3a. Language detection

Check for these marker files (in priority order):

| File | Language/Runtime |
|------|-----------------|
| `package.json` | JavaScript/TypeScript (Node.js) |
| `tsconfig.json` | TypeScript |
| `Cargo.toml` | Rust |
| `go.mod` | Go |
| `requirements.txt`, `pyproject.toml`, `setup.py`, `Pipfile` | Python |
| `Gemfile` | Ruby |
| `pom.xml`, `build.gradle` | Java |
| `mix.exs` | Elixir |
| `composer.json` | PHP |
| `Package.swift` | Swift |
| `*.csproj`, `*.sln` | C# / .NET |
| `Makefile` (alone) | C/C++ |
| `deno.json`, `deno.jsonc` | Deno |
| `bun.lockb` | Bun runtime |

### 3b. Framework detection

If `package.json` exists, read it for:
- `next` -> Next.js
- `react` -> React
- `vue` -> Vue
- `svelte` -> SvelteKit
- `express` -> Express
- `fastify` -> Fastify
- `hono` -> Hono
- `@angular/core` -> Angular
- `astro` -> Astro
- `nuxt` -> Nuxt

If Python, check for:
- `django` -> Django
- `flask` -> Flask
- `fastapi` -> FastAPI
- `streamlit` -> Streamlit

If Rust, check `Cargo.toml` for:
- `actix-web` -> Actix
- `axum` -> Axum
- `rocket` -> Rocket
- `tauri` -> Tauri

### 3c. Package manager detection

| Lock file | Package manager |
|-----------|----------------|
| `bun.lockb` | bun |
| `pnpm-lock.yaml` | pnpm |
| `yarn.lock` | yarn |
| `package-lock.json` | npm |
| `Cargo.lock` | cargo |
| `go.sum` | go modules |
| `Pipfile.lock` | pipenv |
| `poetry.lock` | poetry |
| `uv.lock` | uv |
| `Gemfile.lock` | bundler |

### 3d. Runtime / version detection

```bash
cd "$TARGET"
# Node version
cat .nvmrc .node-version .tool-versions 2>/dev/null | head -1
# Python version
cat .python-version 2>/dev/null
# Rust edition
grep 'edition' Cargo.toml 2>/dev/null
# Go version
grep '^go ' go.mod 2>/dev/null
```

---

## Phase 4: Documentation Audit

Check for the existence and quality of key documentation files.

### 4a. File existence check

Check the repo root for each of these files. Record exists/missing:

| File | Purpose | Status |
|------|---------|--------|
| `README.md` | Project overview | ? |
| `CLAUDE.md` | AI dev commands | ? |
| `ARCHITECTURE.md` | Technical decisions | ? |
| `ETHOS.md` | Builder philosophy | ? |
| `CHANGELOG.md` or `CHANGELOG` | Release notes | ? |
| `VERSION` | Version tracking | ? |
| `TODOS.md` | Work items | ? |
| `CONTRIBUTING.md` | Contribution guide | ? |
| `LICENSE` or `LICENSE.md` | License | ? |
| `DESIGN.md` | Design system | ? |
| `.env.example` | Environment template | ? |

### 4b. README quality check

If README.md exists:
- Does it have a project description?
- Does it have install instructions?
- Does it have usage examples?
- Is it longer than 20 lines? (minimum viable README)
- Does it have badges or status indicators?

### 4c. Count total markdown files

```bash
cd "$TARGET"
find . -name '*.md' -not -path './.git/*' -not -path './node_modules/*' | wc -l
```

---

## Phase 5: Skill & Agent Detection

Find existing AI skill and agent configurations.

### 5a. Skill files

```bash
cd "$TARGET"
# SKILL.md files
find . -name 'SKILL.md' -not -path './.git/*' -not -path './node_modules/*'
# Skill templates
find . -name 'SKILL.md.tmpl' -not -path './.git/*' -not -path './node_modules/*'
```

### 5b. Claude Code configuration

```bash
cd "$TARGET"
# .claude directory
ls -la .claude/ 2>/dev/null
# Commands
ls -la .claude/commands/ 2>/dev/null
# Settings
cat .claude/settings.json 2>/dev/null
cat .claude/settings.local.json 2>/dev/null
```

### 5c. Other agent configurations

```bash
cd "$TARGET"
# OpenAI Codex
ls .agents/ .codex/ agents/ 2>/dev/null
# Amazon Kiro
ls .kiro/ 2>/dev/null
# Cursor
ls .cursor/ .cursorrules 2>/dev/null
# GitHub Copilot
ls .github/copilot-instructions.md 2>/dev/null
```

### 5d. Skill quality signals

For each SKILL.md found, quickly check:
- Has YAML frontmatter?
- Has phases?
- Has verification step?
- Has completion status?
- Has voice directive?
- Word count (rough complexity indicator)

---

## Phase 6: Infrastructure Detection

Check for testing, CI/CD, containerization, and build tooling.

### 6a. Test files

```bash
cd "$TARGET"
# Count test files by pattern
echo "Test files:"
find . -type f \( \
  -name '*.test.*' -o \
  -name '*.spec.*' -o \
  -name 'test_*' -o \
  -name '*_test.go' -o \
  -name '*_test.rs' \
\) -not -path './.git/*' -not -path './node_modules/*' | wc -l

# Test directories
ls -d test/ tests/ spec/ __tests__/ 2>/dev/null
```

### 6b. CI/CD configuration

| File/Directory | CI System |
|---------------|-----------|
| `.github/workflows/` | GitHub Actions |
| `.gitlab-ci.yml` | GitLab CI |
| `.circleci/config.yml` | CircleCI |
| `Jenkinsfile` | Jenkins |
| `.travis.yml` | Travis CI |
| `bitbucket-pipelines.yml` | Bitbucket Pipelines |
| `.buildkite/` | Buildkite |
| `vercel.json` | Vercel |
| `netlify.toml` | Netlify |
| `fly.toml` | Fly.io |
| `render.yaml` | Render |
| `railway.json` | Railway |

```bash
cd "$TARGET"
# List CI configs found
ls .github/workflows/*.yml .github/workflows/*.yaml 2>/dev/null
ls .gitlab-ci.yml .circleci/config.yml Jenkinsfile .travis.yml 2>/dev/null
ls vercel.json netlify.toml fly.toml render.yaml 2>/dev/null
```

### 6c. Containerization

```bash
cd "$TARGET"
ls Dockerfile docker-compose.yml docker-compose.yaml 2>/dev/null
ls *.Dockerfile 2>/dev/null
```

### 6d. Linting and formatting

```bash
cd "$TARGET"
ls .eslintrc* .eslintrc.* eslint.config.* 2>/dev/null
ls .prettierrc* prettier.config.* 2>/dev/null
ls .editorconfig biome.json biome.jsonc 2>/dev/null
ls .stylelintrc* 2>/dev/null
ls .rubocop.yml 2>/dev/null
ls .flake8 .pylintrc pyproject.toml 2>/dev/null  # (pyproject.toml may have [tool.ruff])
ls clippy.toml rustfmt.toml 2>/dev/null
ls .golangci.yml 2>/dev/null
```

### 6e. Build scripts

```bash
cd "$TARGET"
# package.json scripts
cat package.json 2>/dev/null | grep -A 30 '"scripts"' | head -35
# Makefile targets
grep '^[a-zA-Z_-]*:' Makefile 2>/dev/null | head -20
# Other build files
ls justfile Taskfile.yml Rakefile 2>/dev/null
```

---

## Phase 7: Git Analysis

Analyze commit patterns, contributors, and branch structure. Skip this phase entirely if the target is not a git repo.

### 7a. Recent commits

```bash
cd "$TARGET"
git log --oneline -20 2>/dev/null
```

### 7b. Commit message style

Determine the dominant pattern from recent commits:
- Conventional commits (`feat:`, `fix:`, `chore:`)
- Imperative mood (`Add feature`, `Fix bug`)
- Past tense (`Added feature`, `Fixed bug`)
- Prefix style (`[feature]`, `[bugfix]`)
- No pattern (freeform)

### 7c. Contributors

```bash
cd "$TARGET"
git shortlog -sn --all 2>/dev/null | head -10
```

### 7d. Branch structure

```bash
cd "$TARGET"
echo "Current branch: $(git branch --show-current 2>/dev/null)"
echo "Total branches: $(git branch -a 2>/dev/null | wc -l)"
echo "Default branch: $(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's|refs/remotes/origin/||')"
```

### 7e. Activity

```bash
cd "$TARGET"
echo "First commit: $(git log --reverse --format='%ai' 2>/dev/null | head -1)"
echo "Last commit: $(git log -1 --format='%ai' 2>/dev/null)"
echo "Total commits: $(git rev-list --count HEAD 2>/dev/null)"
```

---

## Phase 8: Report

Compile all findings into a structured analysis report.

```
REPO ANALYSIS REPORT
================================================================
Target:          [repo path or URL]
Analyzed:        [timestamp]
================================================================

STRUCTURE
---------
Root dirs:       [list of top-level directories]
Total files:     [count]
Repo size:       [size]
Top extensions:  [.ts: 42, .tsx: 31, .json: 12, ...]

TECH STACK
----------
Language:        [primary language]
Framework:       [detected framework]
Package manager: [detected PM]
Runtime:         [detected runtime + version]

DOCUMENTATION
-------------
[table of doc files: exists/missing]
README quality:  [brief assessment]
Total .md files: [count]

SKILLS & AGENTS
---------------
SKILL.md files:  [count, list paths]
.claude/ dir:    [exists/missing, contents summary]
Other agents:    [list detected agent configs]

INFRASTRUCTURE
--------------
Test files:      [count]
Test framework:  [detected framework]
CI/CD:           [detected system(s)]
Docker:          [yes/no]
Linting:         [list detected tools]

GIT
---
Commits:         [total count]
Contributors:    [count]
Commit style:    [detected pattern]
Age:             [first commit -> last commit]
Default branch:  [branch name]

================================================================
Status:          DONE
Next step:       Run /gf-audit to score this repo against
                 G-Stack quality standards.
================================================================
```

---

## Verification

**IRON LAW: NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE.**

1. Confirm the report covers all 7 analysis areas (structure, stack, docs, skills, infra, git, summary)
2. Confirm no files were created or modified in the target repo
3. Confirm every claim in the report is backed by a command that was actually run
4. If any phase produced errors, note them in the report rather than hiding them

**Rationalization prevention:**
- "Should have detected it" -> RUN THE CHECK.
- "Most repos have X" -> Check this specific repo.
- "I already checked" -> Show the command output.

---

## Completion Status

- **DONE** -- All 8 phases completed. Report produced with evidence.
- **DONE_WITH_CONCERNS** -- Report produced but some phases had issues (noted in report).
- **BLOCKED** -- Cannot proceed. Target path invalid, git clone failed, or permissions denied.
- **NEEDS_CONTEXT** -- Missing target path or URL. Exactly what's needed: the repo to analyze.

### Escalation
- 3 failed attempts at any phase -> STOP, note the failure, continue to next phase
- Uncertain about file classification -> Report what you see, do not guess
- Bad analysis is worse than no analysis

---

## Important Rules

- Never modify, create, or delete any file in the target repo. Read only.
- Never skip a phase. If a phase is not applicable (e.g., no git), say so and move on.
- Never guess at tech stack. If the marker files do not exist, report "not detected."
- Never conflate what exists with what should exist. Report facts, not recommendations (that is /gf-audit's job).
- Always exclude .git/, node_modules/, dist/, build/, venv/, vendor/, target/ from file counts and tree listings.
- If the user asks for a shallow analysis (`--shallow`), run only Phases 1-3 and produce a summary report.
- After completion, suggest running /gf-audit for scoring and recommendations.
- 3 consecutive failures in any phase -> skip that phase, note the failure, continue.
