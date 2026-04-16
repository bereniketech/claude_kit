---
name: generate-claude-md
description: Bootstrap a new project end-to-end. Runs 6 phases in order without stopping — gather context, scaffold infrastructure (git, .gitignore, .claudeignore, .env.example, .claude/, .spec/, .kit/), write stub files, validate, route through route-agents + @company-coo, then lead CEO copies board-selected kit content into .kit/ and runs the company planning skill. CLAUDE.md is a tiny navigation stub — it never lists skills. Skills, agents, commands, and rules are declared per-task inside each task file, so every task is self-sufficient.
---

# Project Bootstrap

This skill is not about generating a file. It bootstraps a project: scaffolds infrastructure, routes through the board, lets the lead CEO select and install kit content, and runs planning so every task file is self-sufficient.

**Hard rule — do not stop early:** Writing files in Phases 2–3 is not the end. You are only done after Phase 6 — planning complete, task files written, kit content copied. If you find yourself thinking "I wrote CLAUDE.md, I'm done" — you are not done. Continue.

**Hard rule — do not analyze into CLAUDE.md:** CLAUDE.md is a fixed tiny navigation stub. Never summarize the project's architecture, stack, or plan into it. Your analysis from Phase 1 is input to the **board** (Phase 5), not to CLAUDE.md.

**Hard rule — skills are task-scoped, never project-scoped:** CLAUDE.md does **not** list skills. Each task file in `.spec/tasks/` declares the exact skills, agents, commands, and rules it needs. Tasks load their own context. This keeps context per-task minimal and every task independently runnable.

---

## Phase 1 — Gather Context

If the user has not described the project, ask one question:

> "What are you building? (e.g. web app, data pipeline, CLI tool, research workflow, document automation)"

If files already exist in the current directory, run `Glob` / `Grep` to infer the project type before asking.

### Confirm PROJECT_ROOT

- If the CWD contains `.claude/skills/*/SKILL.md`, it is the kit library. **Do NOT write project files here.** Ask: "Where should I create the project folder? (e.g. `C:\Users\Hp\Desktop\Experiment\My Project`)"
- If the CWD has existing project files (source code, `package.json`, `pyproject.toml`, the project's own `idea.md`, etc.) matching the description, use CWD as `PROJECT_ROOT`.
- Otherwise ask.

Create the folder if it does not exist. All subsequent file paths are relative to `PROJECT_ROOT`.

### Record project analysis for the board

Capture (in your working memory, not in a file):
- What the project is
- Primary users / outcome
- Any existing files (idea.md, design spec, README, stub code)

This becomes the brief for the board in Phase 5. **Do not write this analysis into any file yet.**

---

## Phase 2 — Scaffold Infrastructure

Set up the project folder so it is git-compliant and ready for the board handoff.

### 2a — Git init

If `git rev-parse --is-inside-work-tree` fails in the project root:
```bash
cd "PROJECT_ROOT" && git init && git branch -M main
```
Otherwise skip.

### 2b — .gitignore

Write `PROJECT_ROOT/.gitignore`:
```
# Environment & secrets
.env
.env.*
!.env.example

# Intermediates
.tmp/

# OS files
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
*.swo

# Logs
*.log
```

If the project type is known, append dependency/build paths:
- Node/TS: `node_modules/`, `dist/`, `build/`
- Python: `__pycache__/`, `.venv/`, `*.pyc`, `dist/`, `*.egg-info/`
- Rust: `target/`
- Java/Kotlin: `target/`, `build/`, `.gradle/`
- Go: `bin/`, `vendor/`

### 2c — .claudeignore

Write `PROJECT_ROOT/.claudeignore`:
```
# Kit content — copied locally, loaded on demand only
.kit/

# Planning artifacts — read only when task requires
.spec/

# Dependencies
node_modules/
__pycache__/
.venv/
target/

# Build output
dist/
build/

# Secrets
.env
.env.*
```

The `.kit/` entry is mandatory — kit files are large and must not auto-load into context.

### 2d — .env.example

Write `PROJECT_ROOT/.env.example` as a stub. If known env vars came from the project description, add them with placeholder values. Otherwise write a minimal placeholder:
```
# Add project environment variables here as they are needed.
# Never commit real secrets to this file.
```

### 2e — Create folders

```bash
mkdir -p "PROJECT_ROOT/.claude"
mkdir -p "PROJECT_ROOT/.spec"
mkdir -p "PROJECT_ROOT/.kit"
```

`.kit/` stays empty until Phase 6a — the board decides what goes in.

**Do not create `.claude/skills/`.** Skills live in `.kit/skills/` after the board selects them.

---

## Phase 3 — Write Stub Files

### 3a — CLAUDE.md (tiny navigation stub)

Write `PROJECT_ROOT/.claude/CLAUDE.md` with **exactly this content**. Do not add sections. Do not summarize the project. Do not list skills — skills are declared per-task.

```markdown
# [Project Name]

## Start Here
1. Read the current task file listed under `## Active Task` below.
2. The task file lists the skills, agents, commands, and rules it needs — load only those.
3. Implement. Run `/task-handoff` when done.

## Active Task
Tasks: `.spec/tasks/` · Current: `.spec/tasks/task-001.md`

## Routing
Need help outside the current task? `@company-coo` — entry point for board routing.
Kit content: `.kit/` (agents · commands · hooks · contexts · rules · skills).

## Bug Log
`bug-log.md` — date · what broke · root cause · fix · files.
```

Replace `[Project Name]` with the actual name. Everything else stays as-is. No `## Skills` section ever. Each task file is self-sufficient.

### 3b — project-config.md

Ask once if not known: "What is the absolute path to your claude_kit directory? (e.g. `C:/Users/Hp/Desktop/Experiment/claude_kit`)"

Store as `KIT_PATH`. Use forward slashes.

Write `PROJECT_ROOT/.claude/project-config.md`:
```markdown
# Project Configuration

## Skill Library
Path: KIT_PATH

## Selected Skills
_(To be populated by @company-coo)_

## Selected Agents
_(To be populated by @company-coo)_

## Selected Commands
_(To be populated by @company-coo)_

## Rules Active
_(To be populated by @company-coo)_

## Stack
_(To be decided by the board)_
```

---

## Phase 4 — Validate

Check each expected path exists:

**Must exist as files:**
- `.git/` directory
- `.gitignore`
- `.claudeignore`
- `.env.example`
- `.claude/CLAUDE.md`
- `.claude/project-config.md`

**Must exist as empty folders:**
- `.claude/`
- `.spec/`
- `.kit/`

If anything is missing, create it now. If something fundamental fails (e.g. git init fails), report the error and stop. Otherwise continue to Phase 5 immediately.

**Reminder:** Writing the files above is **not** the end of the skill. Continue to Phase 5.

---

## Phase 5 — Route Through the Board

This is where project-specific decisions get made. You do not make them — the board does.

### 5a — Classify with route-agents

Invoke the `Skill` tool with `route-agents`, passing the project description captured in Phase 1. `route-agents` returns: the correct board entry point, the lead company (software / marketing / media), and supporting companies.

### 5b — Hand off to @company-coo

Invoke `@company-coo` with:

1. **Project description** — from Phase 1
2. **Existing artifacts** — if the project already has an idea.md, design spec, README, or stub code, mention them by path
3. **Infrastructure status** — git initialized, `.claude/`, `.spec/`, `.kit/` created (kit empty, awaiting board content selection), validation passed
4. **route-agents result** — lead company and supporting companies
5. **Explicit instruction:** "Select the lead CEO. Return the exact list of: (a) skills, (b) agents, (c) commands, (d) rules to copy from KIT_PATH into .kit/. Hooks and contexts can be copied in full — they are small. Then delegate planning to the lead CEO."

Do not invoke a company CEO directly. Always route through `@company-coo`.

---

## Phase 6 — Lead CEO Closes the Loop

Once `@company-coo` returns the selected-content list and hands off to the lead CEO, you (acting as the lead CEO by way of `@company-coo`'s delegation) execute 6a–6e in order.

### 6a — Copy board-selected content into .kit/

Only copy what the board explicitly listed. Agents/commands/skills/rules are selective. Hooks and contexts copy in full (always small, always useful).

```bash
# For each selected skill:
xcopy /E /I /Y "KIT_PATH\skills\<category>\<skill>"    "PROJECT_ROOT\.kit\skills\<category>\<skill>"

# For each selected agent:
xcopy /E /I /Y "KIT_PATH\agents\<company>\<agent>.md"  "PROJECT_ROOT\.kit\agents\<company>\<agent>.md"

# For each selected command:
xcopy /E /I /Y "KIT_PATH\commands\<category>\<cmd>.md" "PROJECT_ROOT\.kit\commands\<category>\<cmd>.md"

# For each selected rule set:
xcopy /E /I /Y "KIT_PATH\rules\<ruleset>"              "PROJECT_ROOT\.kit\rules\<ruleset>"

# Hooks and contexts — always full:
xcopy /E /I /Y "KIT_PATH\hooks"    "PROJECT_ROOT\.kit\hooks"
xcopy /E /I /Y "KIT_PATH\contexts" "PROJECT_ROOT\.kit\contexts"
```

**Rule:** Never modify any file under `KIT_PATH`. The kit is read-only source.

### 6b — Record selections in project-config.md

Update `PROJECT_ROOT/.claude/project-config.md`, filling in the `## Selected Skills`, `## Selected Agents`, `## Selected Commands`, and `## Rules Active` sections with the board's decisions.

### 6c — Lead CEO runs the company planning skill

The lead CEO **immediately** invokes their company's planning skill:

| Lead company | Planning skill |
|---|---|
| software-company | `planning-specification-architecture-software` |
| marketing-company | `planning-specification-architecture-marketing` |
| media-company | `planning-specification-architecture-media` |

The planning skill runs its gated phases (brief/requirements → design → tasks). Each phase requires explicit user approval. Planning artifacts land in `.spec/`.

If the project already had a design spec (e.g. `idea.md`, existing `design.md`), pass it to the planning skill as prior input — treat the run as a revision, not a blank slate.

**Critical instruction to the planning skill:** Every task file in `.spec/tasks/` **must be self-sufficient**. Each task file declares — in its own header — the exact skills, agents, commands, and rules it requires, using plain `.kit/...` paths. A task file is the single source of truth for running that task — no dependency on CLAUDE.md, no dependency on project-wide skill lists.

Task file header format (planning skill enforces this):

```markdown
# Task NNN: <short title>

## Skills
- .kit/skills/<category>/<skill>/SKILL.md
- .kit/rules/<ruleset>/<rule>.md

## Agents
- @<agent-name>

## Commands
- /<command>

## Acceptance Criteria
...

## Steps
...
```

### 6d — Stack decision

After planning, the lead CEO updates the `## Stack` section of `PROJECT_ROOT/.claude/project-config.md` with the decided stack (language, framework, hosting, database, auth).

### 6e — Verify task files are self-sufficient

Before finishing, sanity-check `.spec/tasks/task-001.md` (or whichever tasks the planning skill produced):
- Each task file has its own `## Skills`, `## Agents`, `## Commands` sections at the top
- Paths are plain `.kit/...` references (no `@` imports, no `KIT_PATH`)
- Every referenced `.kit/skills/...` file actually exists inside `.kit/` (copied in 6a)

If a task references a skill/agent/command that was not copied in 6a, go back: copy it into `.kit/` from `KIT_PATH` and update `project-config.md`.

**This is the last action of the skill.** The project is now ready for execution — open any task file and it tells you exactly what context to load.

---

## Re-running on Existing Projects

If the project already has `.claude/project-config.md`:

1. Read `project-config.md` to get `KIT_PATH` — do not ask again.
2. Read existing `.spec/` artifacts.
3. Re-run Phase 5 (route through the board) with the existing `.spec/` content as context — the lead CEO's planning skill treats it as a revision, preserving completed tasks.
4. When the planning skill produces new/revised task files, their self-sufficient headers may reference skills not yet in `.kit/`. Copy any missing items from `KIT_PATH` into `.kit/` and update `project-config.md`.
5. Never edit CLAUDE.md — it is the same tiny stub regardless of how many skills the project uses. Task files carry the skill references, not CLAUDE.md.

---

## Global Rules

### Hierarchy
- Always route: `route-agents` → `@company-coo` → lead CEO → specialists.
- Never invoke a company CEO directly. Never bypass the board.
- Never let the board or COO create plans — planning belongs to the lead CEO's planning skill.

### File management
- Never write into `KIT_PATH`. The kit is read-only source.
- Never copy all of `KIT_PATH/agents/`, `KIT_PATH/commands/`, `KIT_PATH/skills/`, or `KIT_PATH/rules/` in bulk. Only board-selected items. Exception: hooks and contexts copy in full.
- Never use `@` imports in any generated file. All references are plain `.kit/...` paths relative to the project root.
- Never create `.claude/skills/` — skills live in `.kit/skills/`.
- Never write project analysis, architecture, or stack into CLAUDE.md. CLAUDE.md is the fixed tiny stub from Phase 3a.

### Skills are task-scoped
- CLAUDE.md has no `## Skills` section. Ever.
- Each task file in `.spec/tasks/` declares its own skills, agents, commands, and rules in its header.
- A session loads CLAUDE.md → reads current task path → opens the task file → the task file tells it what `.kit/` content to pull in.
- Context per task stays minimal because only task-relevant skills load.

### Flow control
- Writing files in Phases 2–3 is not the end. Continue to Phase 4.
- Phase 4 validation passing is not the end. Continue to Phase 5.
- Phase 5 invoking `route-agents` is not the end. Continue to Phase 6.
- Phase 6 is the end — and only after 6e confirms task files are self-sufficient.
- If any phase fails, report the exact step and the error. Do not silently skip.
