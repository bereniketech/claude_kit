# CLAUDE.md

This file provides guidance to Claude Code when working inside this repository.

## Project Overview

This is a **Claude Agent Skill Library** — a distributable collection of skills, agents, commands, rules, hooks, contexts, and MCP configs. It is a pure library (no build system, no test runner). Content here is installed into other projects, not used in this repo directly.

---

## Repository Structure

```
claude_kit/
├── .claude/
│   ├── package-manager.json              # Package manager config (npm)
│   ├── homunculus/instincts/inherited/   # Curated instincts for this repo
│   └── skills/claude-kit/SKILL.md    # Meta-skill for working in this repo
│
├── skills/          # 53 skill modules organized by category
│   ├── core/            # generate-claude-md, continuous-learning, eval-harness, ...
│   ├── development/     # code-writing, build-website, api-design, ...
│   ├── planning/        # planning-specification, autonomous-agents, multi-model
│   ├── testing-quality/ # tdd-workflow, security-review, security-scan, e2e-testing
│   ├── data-backend/    # postgres-patterns, database-migrations, clickhouse-io
│   ├── languages/       # golang, python, kotlin, java, swift, cpp, android, perl
│   ├── ai-platform/     # claude-developer-platform
│   ├── research-docs/   # research, document-writing, knowledge-management
│   ├── devops/          # terminal-cli-devops
│   ├── ui-design/       # presentations-ui-design
│   ├── integrations/    # x-api, fal-ai, video-editing, videodb, nutrient, visa
│   ├── domain/          # logistics, trade-compliance, energy-procurement
│   └── specialized/     # nanoclaw-repl, ralphinho-rfc-pipeline
├── agents/          # 18 agent definitions, organized by category
│   ├── core/            # planner, architect, chief-of-staff, loop-operator, harness-optimizer
│   ├── development/     # code-reviewer, refactor-cleaner, doc-updater, build-error-resolver
│   ├── testing-quality/ # tdd-guide, security-reviewer, e2e-runner
│   ├── data-backend/    # database-reviewer
│   └── languages/       # go-reviewer, go-build-resolver, kotlin-reviewer, kotlin-build-resolver, python-reviewer
├── commands/        # 49 slash command definitions, organized by category
│   ├── core/            # learn, skill-create, sessions, task-handoff, instinct-*, eval, harness-audit, ...
│   ├── development/     # code-review, build-fix, refactor-clean, verify, update-*, pm2, ...
│   ├── testing-quality/ # tdd, e2e, quality-gate, test-coverage
│   ├── planning/        # plan, orchestrate, model-route, loop-*, multi-*
│   ├── languages/       # go-*, kotlin-*, gradle-build, python-review
│   └── specialized/     # claw
├── rules/           # common/ + 8 language dirs (golang, python, kotlin, ...)
├── contexts/        # dev.md, research.md, review.md
├── hooks/           # hooks.json (PreToolUse, PostToolUse, SessionStart, ...)
├── mcp-configs/     # mcp-servers.json (14 MCP integrations, tokens sanitized)
│
├── everything-claude-code/   # Source repo cloned for reference (do not edit)
└── CLAUDE.md
```

**Design principle:** `.claude/` is minimal — repo meta only. All distributable content lives at root level, mirroring the ECC library architecture.

---

## How to Use This Library

### Zero-Copy Reference Architecture

Projects **never copy files** from this library. Instead, they reference it directly:

- **Skills** → absolute `@` imports in the project's `.claude/CLAUDE.md` (only selected skills load into context)
- **Agents, commands, hooks, contexts, rules** → Windows directory junctions (`mklink /J`) pointing to this kit's directories (zero disk usage, always up to date)

**Result:** Edit any file in the kit → all projects using it see the change automatically on the next session.

### In a new project — generate a full Claude Code setup

From any project directory, ask Claude:

```
/generate-claude-md

I'm building a [description of your project].
```

The `generate-claude-md` skill will:
1. Analyze your project description
2. Select 2–6 relevant skills for the project type
3. Create in your project:
   - `.claude/CLAUDE.md` — with `@` imports for selected skills (absolute kit paths)
   - `.claude/CLAUDE.planning.md` — planning phase guide
   - `.spec/` — planning artifacts
   - `.claude/project-config.md` — kit path + skill list
   - Directory junctions: `.claude/agents/` → kit, `.claude/commands/` → kit, `.claude/rules/common/` → kit, `.claude/hooks/` → kit, `.claude/contexts/` → kit

No files are copied. The kit path is recorded in `project-config.md` for future updates.

### Adding a skill to an existing project

Edit `.claude/CLAUDE.md` and add one line:
```
@C:/Users/Hp/Desktop/Experiment/claude_kit/skills/<category>/<skill-name>/SKILL.md
```
Update `.claude/project-config.md` to record the addition. Active on next session start.

### Load curated instincts for this repo

When working inside the Skill Builder repository itself:

```
/instinct-import .claude/homunculus/instincts/inherited/skill-builder-instincts.yaml
```

---

## Available Content

### Skills (54)

**Core / Meta**
`generate-claude-md` · `new-features-updates` · `skill-stocktake` · `configure-ecc` · `continuous-learning` · `strategic-compact` · `eval-harness`

**Development**
`code-writing-software-development` · `build-website-web-app` · `ide-integration-pair-programming` · `api-design` · `content-hash-cache-pattern`

**Planning & Architecture**
`planning-specification-architecture` · `autonomous-agents-task-automation` · `multi-model-model-agnostic-platforms`

**Testing & Quality**
`tdd-workflow` · `security-review` · `security-scan` · `e2e-testing`

**Data & Backend**
`postgres-patterns` · `database-migrations` · `clickhouse-io`

**Languages**
`golang-patterns` · `python-patterns` · `kotlin-patterns` · `java-patterns` · `swift-patterns` · `cpp-patterns` · `android-patterns` · `perl-patterns`

**Claude / AI Platform**
`claude-developer-platform` · `notebooklm`

**Research & Docs**
`research-information-retreival` · `document-content-writing-editing` · `knowledge-management-productivity`

**DevOps & Infrastructure**
`terminal-cli-devops`

**UI & Design**
`presentations-ui-design`

**Integrations**
`x-api` · `fal-ai-media` · `video-editing` · `videodb` · `nutrient-document-processing` · `visa-doc-translate` · `whatsapp-automation` · `copilot-autonomous-tasks`

**Domain**
`logistics-supply-chain` · `trade-compliance` · `energy-procurement`

**Specialized**
`nanoclaw-repl` · `ralphinho-rfc-pipeline`

### Agents (18)

**Core** — `planner` · `architect` · `chief-of-staff` · `loop-operator` · `harness-optimizer`

**Development** — `code-reviewer` · `refactor-cleaner` · `doc-updater` · `build-error-resolver`

**Testing & Quality** — `tdd-guide` · `security-reviewer` · `e2e-runner`

**Data & Backend** — `database-reviewer`

**Languages** — `go-reviewer` · `go-build-resolver` · `kotlin-reviewer` · `kotlin-build-resolver` · `python-reviewer`

### Commands (49)

**Core** — `learn` · `learn-eval` · `skill-create` · `evolve` · `eval` · `checkpoint` · `save-session` · `resume-session` · `task-handoff` · `sessions` · `aside` · `instinct-export` · `instinct-import` · `instinct-status` · `harness-audit` · `projects` · `promote` · `setup-pm`

**Development** — `code-review` · `build-fix` · `refactor-clean` · `verify` · `update-docs` · `update-codemaps` · `pm2` · `prompt-optimize`

**Testing & Quality** — `tdd` · `e2e` · `quality-gate` · `test-coverage`

**Planning** — `plan` · `orchestrate` · `model-route` · `loop-start` · `loop-status` · `multi-plan` · `multi-execute` · `multi-workflow` · `multi-backend` · `multi-frontend`

**Languages** — `go-build` · `go-review` · `go-test` · `gradle-build` · `kotlin-build` · `kotlin-review` · `kotlin-test` · `python-review`

**Specialized** — `claw`

### Rules
`rules/common/` — 9 files (agents, coding-style, development-workflow, git-workflow, hooks, patterns, performance, security, testing)
`rules/<lang>/` — 5 files each for: golang, kotlin, python, swift, perl, php, typescript

---

## Adding or Updating Skills

1. Create `skills/<category>/<skill-name>/SKILL.md` using the WAT format (see `.claude/skills/claude-kit/SKILL.md`)
2. Add the skill to `skills/core/generate-claude-md/SKILL.md` under the correct category
3. Update the Available Content table in this file
4. Check for overlap with existing skills first — merge if >50% overlap

**WAT format:** YAML frontmatter (`name` + `description` only), numbered sections, `---` separators, second-person imperative, bold `**Rule:**` callouts. Token-minimal — no preamble, no redundancy.
