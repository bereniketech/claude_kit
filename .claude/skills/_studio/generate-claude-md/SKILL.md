---
name: project-bootstrap
description: Full project bootstrap — 6 mandatory phases executed in strict order. Phase 0 detects prior run. Phase 1 gathers context. Phase 2 scaffolds git + ignore files + folders. Phase 3 writes fixed CLAUDE.md stub (10 lines only — no analysis, no stack) + project-config.md. Phase 4 validates. Phase 5 selects kit content. Phase 6 copies kit into .kit/ then reads and executes the planning skill in-context. DONE only when .spec/tasks/task-001.md exists. Writing CLAUDE.md is Phase 3 — never Phase 1.
---

# Project Bootstrap

## HOW TO RUN THIS SKILL

**Do not execute this skill inline. Spawn it as an Agent.**

When you read this file, your only job is to call the `Agent` tool with the full content of this skill as the prompt, plus the user's project description. Everything below (Phases 0–6) runs inside that agent. Do not write any files yourself. Do not read `idea.md` yourself. Do not generate a CLAUDE.md yourself. Spawn the agent and stop.

```
Agent(
  description: "Project bootstrap — full 6-phase scaffold + planning",
  prompt: "<paste full skill content here, then append:>

PROJECT_ROOT: <absolute path to project folder>
KIT_PATH: C:/Users/Hp/Desktop/Experiment/claude_kit
User description: <what the user said they are building>
Existing files: idea.md exists at PROJECT_ROOT (read it in Phase 1)"
)
```

---

## AGENT ENTRY POINT

If you are running as the spawned agent, continue below. Execute all phases in order without stopping.

> **HARD STOP — read before doing anything else.**
> Your first action is NOT to write a file. Your first action is Phase 0 — detect prior run state.
> Writing `CLAUDE.md` — at root OR at `.claude/CLAUDE.md` — is **Phase 3, not Phase 1**.
> Do not call `Write` until Phase 2 infrastructure is complete.

---

## Completion Contract

You are NOT done until ALL of these exist on disk:

| # | Path | Notes |
|---|---|---|
| 1 | `PROJECT_ROOT/.git/` | git repo |
| 2 | `PROJECT_ROOT/.gitignore` | |
| 3 | `PROJECT_ROOT/.claudeignore` | must include `.kit/` and `.spec/` |
| 4 | `PROJECT_ROOT/.env.example` | |
| 5 | `PROJECT_ROOT/.claude/CLAUDE.md` | exactly the fixed stub — no analysis, no stack, no skills |
| 6 | `PROJECT_ROOT/.claude/project-config.md` | all 5 sections populated: Skills, Agents, Commands, Rules, Stack |
| 7 | `PROJECT_ROOT/.kit/` | contains board-selected skills, agents, commands, rules + full hooks/ + contexts/ |
| 8 | `PROJECT_ROOT/.spec/` | planning artifacts |
| 9 | `PROJECT_ROOT/.spec/tasks/task-001.md` | has its own ## Skills, ## Agents, ## Commands sections |

At Phase 6e, walk this list. If anything is missing, fix it before reporting done.

---

## Three Absolute Rules

**Rule 1 — Never stop at Phase 3.** Writing CLAUDE.md does not end the skill. Continue to Phase 4, 5, 6.

**Rule 2 — CLAUDE.md is the fixed stub, always.** Never put architecture, tech stack, pipeline descriptions, or skill lists in CLAUDE.md. If a CLAUDE.md exists with that content, overwrite it with the stub. Stack and architecture go in `.spec/design.md`. Skills go in task files.

**Rule 3 — CLAUDE.md has no `## Skills` section. Ever.** Skills are per-task. Each `.spec/tasks/*.md` file declares its own.

---

## Phase 0 — Detect Prior Run

Check `PROJECT_ROOT` and classify:

| Observed | Action |
|---|---|
| No `.claude/` folder | Start at Phase 1 |
| `CLAUDE.md` at project root with analysis content | Delete it. Write correct stub at `.claude/CLAUDE.md` in Phase 3 |
| `.claude/CLAUDE.md` has `## Stack`, `## Architecture`, or `## Skills` | Overwrite with fixed stub in Phase 3 |
| `.claude/project-config.md` populated + `.kit/` populated + `.spec/tasks/` exists | Jump to Phase 6e (verify only) |
| Partial state (some files exist, tasks missing) | Resume from first incomplete phase |

State the classification in one sentence. Then continue — never stop here.

---

## Phase 1 — Gather Context

If `idea.md`, `README.md`, or source files exist, read them — do not ask the user what they're building.

If nothing exists, ask once: "What are you building?"

### Confirm PROJECT_ROOT and KIT_PATH

- CWD has `.claude/skills/*/SKILL.md` → this is the kit itself. Ask for the project path.
- CWD has project files matching the description → use CWD.
- Otherwise ask once.

Ask if not already known: "Absolute path to claude_kit? (default: `C:/Users/Hp/Desktop/Experiment/claude_kit`)" — store as `KIT_PATH`. Use forward slashes. Do not ask again in Phase 3b.

### Build the project brief (working memory only — do not write to any file)

- Project name + one-line purpose
- Primary users / outcome
- Existing artifacts by path
- Tech stack hints from existing files

**NEXT: Phase 2.**

---

## Phase 2 — Scaffold Infrastructure

### 2a — Git

If `.git/` missing: `git init && git branch -M main`

### 2b — .gitignore

```
# Environment & secrets
.env
.env.*
!.env.example

# Intermediates
.tmp/

# OS
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

Add stack-specific entries (Node: `node_modules/ dist/ build/` · Python: `__pycache__/ .venv/ *.pyc dist/ *.egg-info/` · Rust: `target/` · Java/Kotlin: `target/ build/ .gradle/` · Go: `bin/ vendor/`)

### 2c — .claudeignore

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

`.kit/` is mandatory — kit files must not auto-load.

### 2d — .env.example

Stub with known env vars from the project brief. If none known:
```
# Add project environment variables here as they are needed.
# Never commit real secrets to this file.
```

### 2e — Folders

```bash
mkdir "PROJECT_ROOT/.claude" "PROJECT_ROOT/.spec" "PROJECT_ROOT/.kit"
```

Never create `.claude/skills/` — skills live in `.kit/skills/`.

**NEXT: Phase 3.**

---

## Phase 3 — Write Stub Files

### 3a — CLAUDE.md (fixed — do not deviate)

Write `PROJECT_ROOT/.claude/CLAUDE.md` with EXACTLY this. Replace only `[Project Name]`. Add nothing else.

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

If `PROJECT_ROOT/CLAUDE.md` (root level) exists, delete it after writing the correct one at `.claude/CLAUDE.md`.

### 3b — project-config.md

Use `KIT_PATH` confirmed in Phase 1. Do not ask again.

Write `PROJECT_ROOT/.claude/project-config.md`:

```markdown
# Project Configuration

## Skill Library
Path: KIT_PATH

## Selected Skills
_(populated in Phase 5)_

## Selected Agents
_(populated in Phase 5)_

## Selected Commands
_(populated in Phase 5)_

## Rules Active
_(populated in Phase 5)_

## Stack
_(populated in Phase 6)_
```

**NEXT: Phase 4.**

---

## Phase 4 — Validate

Check every path in the Completion Contract rows 1–6. Create any missing item now. If git init fails, report and stop. Otherwise continue.

Re-read `.claude/CLAUDE.md` — if it has any content beyond the fixed stub, rewrite it now.

**NEXT: Phase 5 — do not stop here.**

---

## Phase 5 — Select Kit Content (YOU do this inline — no sub-agents)

Based on the project brief from Phase 1, select the items to install. Use the routing table and selection rules below. Do not delegate this to `@company-coo` or `route-agents` — those are downstream consumers of a bootstrapped project, not inputs to bootstrapping.

### 5a — Identify lead company

| Project type | Lead company | Planning skill path |
|---|---|---|
| Software, product, AI, API, infra, data, security, OS, web app, CLI, pipeline | `software-company` | `KIT_PATH/skills/planning/planning-specification-architecture-software/SKILL.md` |
| SEO, ads, growth, brand, email campaigns | `marketing-company` | `KIT_PATH/skills/planning/planning-specification-architecture-marketing/SKILL.md` |
| Blog, video, podcast, newsletter, social, content, docs | `media-company` | `KIT_PATH/skills/planning/planning-specification-architecture-media/SKILL.md` |

Record `LEAD_COMPANY` and the exact planning skill path for use in Phase 6d.

### 5b — Select skills

**Always include (every project):** `KIT_PATH/skills/core/karpathy-principles/SKILL.md`

Pick 3–8 additional skills from `KIT_PATH/skills/` that directly match the project's tech stack and workload. Use the category structure from CLAUDE.md. Examples for a Python FastAPI + AI project: `languages/python-patterns`, `frameworks-backend/fastapi-patterns`, `data-science-ml/ai-engineer`, `testing-quality/tdd-workflow`, `devops/terminal-cli-devops`.

### 5c — Select agents

Pick the relevant agents from `KIT_PATH/agents/`. Always include:
- `board/company-coo.md`
- `software-company/software-cto.md` (or equivalent CEO for lead company)
- 3–6 specialist agents relevant to the stack

### 5d — Select commands

Pick relevant commands from `KIT_PATH/commands/`. Always include: `core/` (task-handoff, wrapup, code-review). Add stack-specific commands (e.g. `languages/python-review`, `testing-quality/tdd`).

### 5e — Select rules

Always include `rules/common/` in full. Add language-specific rule sets if applicable (e.g. `rules/python/`, `rules/golang/`).

**NEXT: Phase 6.**

---

## Phase 6 — Install Kit Content + Run Planning

### 6a — Copy selected content into .kit/

```bash
# For each selected skill:
xcopy /E /I /Y "KIT_PATH\skills\<category>\<skill>" "PROJECT_ROOT\.kit\skills\<category>\<skill>"

# For each selected agent:
copy "KIT_PATH\agents\<company>\<agent>.md" "PROJECT_ROOT\.kit\agents\<company>\"

# For each selected command:
copy "KIT_PATH\commands\<category>\<cmd>.md" "PROJECT_ROOT\.kit\commands\<category>\"

# Selected rules:
xcopy /E /I /Y "KIT_PATH\rules\<ruleset>" "PROJECT_ROOT\.kit\rules\<ruleset>"

# Always — hooks and contexts in full:
xcopy /E /I /Y "KIT_PATH\hooks"    "PROJECT_ROOT\.kit\hooks"
xcopy /E /I /Y "KIT_PATH\contexts" "PROJECT_ROOT\.kit\contexts"

# Always — rules/common/ in full:
xcopy /E /I /Y "KIT_PATH\rules\common" "PROJECT_ROOT\.kit\rules\common"

# Always — batch-tasks skill (required for multi-task projects):
xcopy /E /I /Y "KIT_PATH\skills\_studio\batch-tasks" "PROJECT_ROOT\.kit\skills\_studio\batch-tasks"
```

Never modify any file under `KIT_PATH`. Kit is read-only source.

### 6b — Update project-config.md

Fill in all five sections (Selected Skills, Selected Agents, Selected Commands, Rules Active, leave Stack for 6e).

### 6c — Verify .kit/ contents

List `.kit/` and confirm every selected file exists. If any is missing, re-copy it now.

### 6d — Execute the planning skill NOW

Read the planning skill file for the lead company (from Phase 5a) and execute it in-context. Do not delegate to a sub-agent. Do not use the Skill tool. Read the file and run it as if its instructions were written here.

| Lead company | File to read |
|---|---|
| `software-company` | `KIT_PATH/skills/planning/planning-specification-architecture-software/SKILL.md` |
| `marketing-company` | `KIT_PATH/skills/planning/planning-specification-architecture-marketing/SKILL.md` |
| `media-company` | `KIT_PATH/skills/planning/planning-specification-architecture-media/SKILL.md` |

Pass into the planning skill execution:
- `PROJECT_ROOT`
- Project brief from Phase 1
- Path to `idea.md` if it exists (treat as prior input — revision run, not blank slate)
- Instruction: every task file must have `## Skills`, `## Agents`, `## Commands` sections at the top using `.kit/...` paths

Task file format the planning skill must emit:

```markdown
# Task NNN: <title>

## Skills
- .kit/skills/<category>/<skill>/SKILL.md

## Agents
- @<agent-name>

## Commands
- /<command>

## Acceptance Criteria
...

## Steps
...
```

The planning skill runs gated phases (brief → design → tasks) with user approval at each gate. Artifacts land in `.spec/`.

### 6e — After planning: verify Completion Contract

After the planning skill returns:

1. Update `## Stack` in `project-config.md` with the stack from the design doc.
2. Walk the Completion Contract table at the top of this skill. Check every row.
3. For every `.spec/tasks/*.md` file: confirm it has `## Skills`, `## Agents`, `## Commands` at the top with `.kit/...` paths. If any task references a `.kit/` item not yet copied, copy it now and add to `project-config.md`.
4. If any Completion Contract row fails, fix it before reporting.

### 6f — Final report (3 lines)

```
Bootstrap complete. Lead: [LEAD_COMPANY].
Kit: N skills, M agents, K commands installed in .kit/.
Planning: .spec/tasks/task-001.md … task-NNN.md written. Open task-001.md to start.
```

**This is the end of the skill.**

---

## Re-Run Behavior

If `project-config.md` already exists and is populated:

1. Read `KIT_PATH` from it — do not ask again.
2. Verify `.claude/CLAUDE.md` is still the fixed stub. Rewrite if it has drifted.
3. Read existing `.spec/` artifacts.
4. Re-run Phase 5 (re-select if needed) and Phase 6d with existing `.spec/` as prior input.
5. Copy any newly-required `.kit/` items missing on disk.

---

## Global Rules

- Never write into `KIT_PATH`.
- Never bulk-copy `KIT_PATH/agents/`, `/commands/`, `/skills/`, `/rules/` in their entirety. Only selected items. Exception: `hooks/`, `contexts/`, `rules/common/` copy in full.
- Never use `@` imports in any generated file. All references are `.kit/...` paths from project root.
- Never create `.claude/skills/` — skills live in `.kit/skills/`.
- Never put project analysis into CLAUDE.md. It goes in `.spec/design.md`.
- Phase boundary reminder: Phase 3 done → continue. Phase 4 done → continue. Phase 5 done → continue. Phase 6d done → verify and report. Never stop between phases.
