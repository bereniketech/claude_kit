---
name: generate-claude-md
description: Bootstrap a project end-to-end. Runs 7 phases in strict order — detect/repair prior run, gather context, scaffold infrastructure, write stub files, validate, route through board, lead CEO copies kit content and runs planning. CLAUDE.md is a fixed tiny navigation stub — never lists skills. Every task file in .spec/tasks/ declares its own skills/agents/commands/rules so tasks are self-sufficient.
---

# Project Bootstrap

This skill does NOT generate a single CLAUDE.md file. It bootstraps an entire project and does not finish until planning is complete, task files exist, and kit content is installed.

---

## Completion Contract — read this first

You are NOT done until ALL of these exist:

1. `PROJECT_ROOT/.git/` directory
2. `PROJECT_ROOT/.gitignore`
3. `PROJECT_ROOT/.claudeignore`
4. `PROJECT_ROOT/.env.example`
5. `PROJECT_ROOT/.claude/CLAUDE.md` (exactly the fixed stub from Phase 3a — no analysis, no stack, no skills section)
6. `PROJECT_ROOT/.claude/project-config.md` with `## Selected Skills / Agents / Commands / Rules Active / Stack` all populated
7. `PROJECT_ROOT/.kit/` contains the board-selected skills, agents, commands, rules (plus full hooks/ and contexts/)
8. `PROJECT_ROOT/.spec/` contains planning artifacts (brief, design, tasks)
9. `PROJECT_ROOT/.spec/tasks/task-001.md` exists with its own `## Skills`, `## Agents`, `## Commands` sections at the top

If any of these are missing, the skill is not complete. Do not stop. Do not return control to the user with "I've created CLAUDE.md". Continue until all 9 checks pass.

---

## Three Hard Rules

1. **Never stop at Phase 3.** Writing CLAUDE.md is not the end. Writing all stub files is not the end. Phase 6 + Phase 7 must run.
2. **Never write project analysis into CLAUDE.md.** CLAUDE.md is the fixed 10-line stub from Phase 3a. Architecture, tech stack, and pipeline descriptions belong in `.spec/design.md`, NEVER in CLAUDE.md. If the user's existing CLAUDE.md contains analysis, overwrite it with the stub.
3. **CLAUDE.md has no `## Skills` section. Ever.** Skills are task-scoped. Each task file in `.spec/tasks/` declares its own skills/agents/commands/rules.

---

## Phase 0 — Detect Prior Run

Before touching anything, inspect `PROJECT_ROOT` to decide entry point.

Run these checks and classify:

| Observed state | Entry phase |
|---|---|
| Empty or only `README.md` / `idea.md` | Start at Phase 1 |
| `CLAUDE.md` exists at project root (wrong location — should be `.claude/CLAUDE.md`) | Read it for context, then **overwrite with stub in Phase 3a** at correct path |
| `.claude/CLAUDE.md` exists but contains `## Stack`, `## Architecture`, or any prose beyond the fixed stub | **Overwrite in Phase 3a** with the correct stub |
| `.claude/project-config.md` exists and is fully populated, `.kit/` populated, `.spec/tasks/` exists | Jump to Phase 7 (re-plan only) |
| Partial: some stubs but `.kit/` empty or `.spec/tasks/` missing | Resume from the first incomplete phase, but re-run Phase 3a to ensure stubs are correct |

State the classification in one line to the user, e.g.: "Detected partial prior run — CLAUDE.md at project root contains full analysis. Will move to `.claude/CLAUDE.md` as fixed stub and continue from Phase 2."

Then continue. Never stop here.

---

## Phase 1 — Gather Context

If the user has not described the project and no `idea.md` exists, ask:

> "What are you building? (e.g. web app, data pipeline, CLI tool, research workflow, document automation)"

If `idea.md`, `README.md`, or stub code exists in `PROJECT_ROOT`, read them to build the project brief — do not re-ask the user.

### Confirm PROJECT_ROOT

- If CWD contains `.claude/skills/*/SKILL.md`, it is the kit library. **Do NOT write project files here.** Ask: "Where should I create the project folder?"
- If CWD has project files matching the user's description, use CWD as `PROJECT_ROOT`.
- Otherwise ask once.

### Build the board brief (in working memory, NOT in a file)

Capture:
- Project name and one-line purpose
- Primary users / outcome
- Existing artifacts (paths only — e.g. `idea.md`, `README.md`)
- Any tech stack hints from existing files

This brief is input to Phase 5. Do not write it into any file. **Especially not CLAUDE.md.**

**NEXT: Phase 2.**

---

## Phase 2 — Scaffold Infrastructure

### 2a — Git init

If `.git/` missing: `cd PROJECT_ROOT && git init && git branch -M main`

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

Append dependency/build paths if project type is known:
- Node/TS: `node_modules/`, `dist/`, `build/`
- Python: `__pycache__/`, `.venv/`, `*.pyc`, `dist/`, `*.egg-info/`
- Rust: `target/`
- Java/Kotlin: `target/`, `build/`, `.gradle/`
- Go: `bin/`, `vendor/`

### 2c — .claudeignore

Write `PROJECT_ROOT/.claudeignore`:

```
.kit/
.spec/
node_modules/
__pycache__/
.venv/
target/
dist/
build/
.env
.env.*
```

The `.kit/` entry is mandatory — kit files are large and must not auto-load.

### 2d — .env.example

Write `PROJECT_ROOT/.env.example`. If env vars can be inferred from the project brief, include them with placeholder values. Otherwise minimal:

```
# Add project environment variables here as they are needed.
# Never commit real secrets to this file.
```

### 2e — Create folders

```bash
mkdir -p "PROJECT_ROOT/.claude" "PROJECT_ROOT/.spec" "PROJECT_ROOT/.kit"
```

`.kit/` stays empty until Phase 6a. **Do not create `.claude/skills/`** — skills live in `.kit/skills/`.

**NEXT: Phase 3.**

---

## Phase 3 — Write Stub Files

### 3a — CLAUDE.md (fixed stub — no deviation)

Write `PROJECT_ROOT/.claude/CLAUDE.md` with EXACTLY the content below. No extra sections. No tech stack. No architecture. No skills list. Replace only `[Project Name]`.

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

**If a CLAUDE.md with different content already exists anywhere in the project (root or `.claude/`), OVERWRITE it with this stub.** Tech stack, architecture, and pipeline details belong in `.spec/design.md`, NEVER in CLAUDE.md. If the old CLAUDE.md had a root-level copy (e.g. `PROJECT_ROOT/CLAUDE.md`), delete that file after writing the correct one at `.claude/CLAUDE.md`.

### 3b — project-config.md

Ask once if unknown: "Absolute path to your claude_kit directory? (default: `C:/Users/Hp/Desktop/Experiment/claude_kit`)". Use forward slashes.

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

**NEXT: Phase 4.**

---

## Phase 4 — Validate

Verify every path exists. If any is missing, create it now. On fundamental failure (e.g. git init failed), report and stop. Otherwise continue immediately.

**Files required:** `.git/`, `.gitignore`, `.claudeignore`, `.env.example`, `.claude/CLAUDE.md`, `.claude/project-config.md`
**Folders required (may be empty):** `.claude/`, `.spec/`, `.kit/`

Reading back `.claude/CLAUDE.md`: it must be exactly the fixed stub. If anything else is in it, rewrite it now.

**NEXT: Phase 5 — do NOT stop here.**

---

## Phase 5 — Route Through the Board

You do not make architecture decisions. The board does.

### 5a — Classify with route-agents

Invoke the `Skill` tool with `route-agents`, passing the project brief from Phase 1. It returns: board entry point, lead company, supporting companies.

### 5b — Invoke @company-coo

Use the `Agent` tool (or `@company-coo` invocation) with this exact brief:

1. **Project description** — from Phase 1
2. **Existing artifacts** — paths to `idea.md`, `README.md`, stub code
3. **Infrastructure status** — "git initialized; `.claude/`, `.spec/`, `.kit/` created; stubs written; validation passed"
4. **route-agents result** — lead company, supporting companies
5. **Required return payload** — "Return a JSON block containing:
   ```
   {
     "lead_company": "software-company | marketing-company | media-company",
     "lead_ceo": "<agent name>",
     "skills": ["<category>/<skill-name>", ...],
     "agents": ["<company>/<agent-name>", ...],
     "commands": ["<category>/<command-name>", ...],
     "rules": ["<ruleset>", ...]
   }
   ```
   Do not create plans. Do not write files. Only return the selection JSON."

Never invoke a company CEO directly. Always route through `@company-coo`.

**NEXT: Phase 6 — YOU (the main thread) execute 6a–6d using the JSON returned by @company-coo. Do NOT wait for the COO to do the copying. Do NOT hand off to the CEO yet.**

---

## Phase 6 — Install Kit Content (YOU do this, not a sub-agent)

You — the main thread running this skill — execute all of 6a–6d. This is not delegated. The board returned a selection; your job is to install it.

### 6a — Copy board-selected content into .kit/

For each item in the JSON returned by @company-coo:

```bash
# Skills (one dir per skill):
xcopy /E /I /Y "KIT_PATH\skills\<category>\<skill>"    "PROJECT_ROOT\.kit\skills\<category>\<skill>"

# Agents (one file per agent):
xcopy /Y "KIT_PATH\agents\<company>\<agent>.md"        "PROJECT_ROOT\.kit\agents\<company>\"

# Commands (one file per command):
xcopy /Y "KIT_PATH\commands\<category>\<cmd>.md"       "PROJECT_ROOT\.kit\commands\<category>\"

# Rules (one dir per ruleset):
xcopy /E /I /Y "KIT_PATH\rules\<ruleset>"              "PROJECT_ROOT\.kit\rules\<ruleset>"

# Hooks + contexts — always full copy:
xcopy /E /I /Y "KIT_PATH\hooks"    "PROJECT_ROOT\.kit\hooks"
xcopy /E /I /Y "KIT_PATH\contexts" "PROJECT_ROOT\.kit\contexts"
```

Also copy `rules/common/` in full if it exists:
```bash
xcopy /E /I /Y "KIT_PATH\rules\common" "PROJECT_ROOT\.kit\rules\common"
```

**Never modify any file under `KIT_PATH`.** The kit is read-only source.

### 6b — Record selections in project-config.md

Update `PROJECT_ROOT/.claude/project-config.md` — fill `## Selected Skills`, `## Selected Agents`, `## Selected Commands`, `## Rules Active` with the exact items the board chose. Leave `## Stack` alone for now (Phase 7d sets it).

### 6c — Verify .kit/ contents match the selection

List `PROJECT_ROOT/.kit/` and confirm every skill/agent/command/rule from the JSON is present as a file. If any is missing, re-copy it. Do not proceed until the kit matches the selection.

### 6d — Confirm completion of Phase 6

Report to the user, one line: "Kit installed: N skills, M agents, K commands, R rulesets. Lead CEO: `<lead_ceo>`. Invoking planning skill now."

**NEXT: Phase 7 — invoke the planning skill immediately. Do NOT stop.**

---

## Phase 7 — Run the Lead CEO's Planning Skill

Invoke the `Skill` tool RIGHT NOW with the correct planning skill for the lead company. This is a direct `Skill` tool call — not a sub-agent, not a description of what should happen. Actually call the tool.

| Lead company | Planning skill argument |
|---|---|
| software-company | `planning-specification-architecture-software` |
| marketing-company | `planning-specification-architecture-marketing` |
| media-company | `planning-specification-architecture-media` |

Pass in the skill input:
- `PROJECT_ROOT`
- Project brief from Phase 1
- Path to `idea.md` if it exists — treat as prior input, run as a revision not a blank slate
- **Mandatory task file header format** (planning skill MUST emit this for every task):

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

The planning skill runs its own gated phases (brief → design → tasks) — each gated on user approval. Artifacts land in `PROJECT_ROOT/.spec/`.

### 7a — After planning returns: stack decision

Update `## Stack` in `PROJECT_ROOT/.claude/project-config.md` with the decided stack (language, framework, hosting, database, auth) from the design doc.

### 7b — Verify task files are self-sufficient

For every file in `.spec/tasks/`:
- Has `## Skills`, `## Agents`, `## Commands` sections at the top
- Uses plain `.kit/...` paths (NO `@` imports, NO `KIT_PATH`)
- Every referenced `.kit/skills/...` and `.kit/agents/...` file actually exists on disk

If a task references a skill not copied in 6a: copy it from `KIT_PATH` now and append it to `## Selected Skills` in `project-config.md`. Repeat until every task file's references resolve.

### 7c — Run the Completion Contract

Walk through all 9 items in the Completion Contract at the top of this file. If any fails, fix it before reporting done.

### 7d — Report

Final report to user, 3 lines max:
1. "Bootstrap complete. Lead: `<lead_ceo>`."
2. "Kit: N skills, M agents, K commands installed under `.kit/`."
3. "Planning: `.spec/tasks/task-001.md` … `.spec/tasks/task-NNN.md` written. Open `task-001.md` to start."

**This — and only this — is the end of the skill.**

---

## Re-Run Behavior

If `PROJECT_ROOT/.claude/project-config.md` already exists and is populated:

1. Read `project-config.md` for `KIT_PATH` — do not ask again.
2. Verify the fixed CLAUDE.md stub at `.claude/CLAUDE.md`. If it drifted (has Stack, Architecture, Skills sections, or is at the wrong location), rewrite it as the stub.
3. Read existing `.spec/` artifacts.
4. Re-run Phase 5 with the existing `.spec/` content as prior input — planning treats it as a revision, not a blank slate.
5. Copy any newly-referenced `.kit/` items not already on disk.
6. CLAUDE.md stays the fixed stub forever. Do not edit it to reflect project state.

---

## Global Rules (reference)

### Hierarchy
- Always route: `route-agents` → `@company-coo` → lead CEO → specialists.
- Never invoke a company CEO directly. Never bypass the board.
- The board/COO selects content; they do NOT create plans. Planning belongs to the lead CEO's planning skill.

### File management
- Never write into `KIT_PATH`. Kit is read-only source.
- Never bulk-copy `KIT_PATH/agents/`, `/commands/`, `/skills/`, or `/rules/`. Only board-selected items. Exception: `hooks/` and `contexts/` copy in full.
- Never use `@` imports in generated files. All references are plain `.kit/...` paths from `PROJECT_ROOT`.
- Never create `.claude/skills/` — skills live in `.kit/skills/`.
- Never put project analysis, architecture, or stack into CLAUDE.md. Those go in `.spec/design.md`. CLAUDE.md is the fixed stub.

### Skills are task-scoped
- CLAUDE.md has no `## Skills` section. Ever.
- Each `.spec/tasks/*.md` declares its own skills/agents/commands/rules.
- Session flow: CLAUDE.md → current task path → open task file → load only that task's `.kit/` items.

### Flow control (the thing the skill keeps failing on)
- Phase 3 writing stubs is NOT the end. Continue to Phase 4.
- Phase 4 validation passing is NOT the end. Continue to Phase 5.
- Phase 5 getting the board selection JSON is NOT the end. Continue to Phase 6.
- Phase 6 copying kit content is NOT the end. Continue to Phase 7.
- Phase 7 is the end — and only after 7c (Completion Contract) passes.
- At every phase boundary the skill has a "NEXT:" directive. Honor it literally. If you find yourself wanting to summarize or return control, re-read the Completion Contract and keep going.
