---
name: generate-claude-md
description: Set up a new project — git, .gitignore, .env.example, .claude/, .spec/, .kit/ — then route through the board to select skills, copy them into .kit/, run the company planning skill, and write a lean CLAUDE.md. Use when starting a new project.
---

# Generate CLAUDE.md Skill

## Step 1 — Gather Context

If the user has not described the project, ask exactly one question:

> "What are you building? (e.g. web app, data pipeline, CLI tool, research workflow, document automation)"

If files exist in the working directory, scan them (`Glob`, `Grep`) to infer the project type before asking.

## Step 1b — Confirm Project Root

**Before writing any file**, determine where the project should live.

### Detect Skill Library CWD
Check if the current working directory is a skill library by testing for `.claude/skills/*/SKILL.md` files. If found, the CWD is the skill library — **do NOT write project files here**.

### Ask for Project Folder
If the CWD is a skill library (or has no relation to the project being described), ask:

> "Where should I create the project folder? (e.g. `C:\Users\Hp\Desktop\Experiment\My Project`)"

Create the folder if it does not exist. Set this as `PROJECT_ROOT` — every file write in all subsequent steps must use this absolute path as the base.

### CWD is Already the Project
If the CWD contains existing project files (source code, `package.json`, `pyproject.toml`, etc.) that match the described project, use CWD as `PROJECT_ROOT` without asking.

### Rule
**All file paths in Steps 4–10 are relative to `PROJECT_ROOT`, not the CWD.** Never write `.claude/`, `.spec/`, `.gitignore`, or `.env.example` into the skill library directory.

## Step 2 — Do NOT Gather Deployment Info (Board Decides)

**Skip all deployment, hosting, stack, and auth decisions.** The board will assess the project and decide the right stack, package manager, hosting, database, and auth. Pre-filling these before the board has seen the project produces wrong defaults that the board then has to undo.

## Step 3 — Analyze Project Description

From Step 1, understand what's being built:
- Type of project (web app, CLI, data pipeline, etc.)
- Primary purpose and users
- Any existing codebase or starting point

This becomes the input to the board in Step 6. **Do not select skills, plan features, or make stack decisions here** — that belongs to the board. Proceed immediately to Step 4 (infrastructure setup).

## Step 4 — Infrastructure Setup

Set up the project folder so it is git-compliant and ready for the board handoff.

The project folder must be git-compliant so it can be pushed directly to a remote.

### 4a — Initialize Git Repository

If the project root is not already a git repo (`Bash: git rev-parse --is-inside-work-tree`):
1. Run `git init` in the project root.
2. Set the default branch to `main` (`git branch -M main`).

If it is already a git repo, skip this step.

### 4b — Generate .gitignore

Create or append to `.gitignore` in the project root. Ensure these entries exist (do not duplicate if already present):

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

# Dependencies (uncomment the one that applies)
# node_modules/
# __pycache__/
# .venv/

# IDE
.vscode/
.idea/
*.swp
*.swo

# Logs
*.log
```

If the project type is known, uncomment the relevant dependency line (e.g., `node_modules/` for JS/TS projects, `__pycache__/` and `.venv/` for Python projects). Add any additional entries appropriate for the detected stack.

### 4c — Create .env.example

If any environment variables are already known from the project description, create `.env.example` with placeholder comments — no real values. Otherwise leave this for the board to define:

```
# Database
DATABASE_URL=your_database_url_here

# Auth
AUTH_SECRET=your_auth_secret_here
```

This file IS committed to git (it's excluded from `.gitignore` via `!.env.example`).

### 4d — Set Up Project Folders

Create `.claude/` in the project root if it doesn't exist. All generated config and CLAUDE.md files go here.
Create `.spec/` in the project root if it doesn't exist. All planning artifacts (`requirements.md`, `design.md`, `tasks/`) go here.

**Do not create `.claude/skills/`** — selected skills are copied into `.kit/skills/` by the lead CEO in Step 6c.

## Step 5 — Copy Kit Content + Write CLAUDE.md

Kit content is **copied** into `PROJECT_ROOT/.kit/` — agents, commands, hooks, contexts, rules, and selected skills. The kit itself is never modified. The project is fully self-contained.

### 5a — Determine Kit Path

Read the absolute path of the skill library. If unknown, ask the user once:
> "What is the absolute path to your claude_kit directory? (e.g. `C:/Users/Hp/Desktop/Experiment/claude_kit`)"

Store this as `KIT_PATH`. Use forward slashes throughout.

### 5b — Create .kit/ Folder

Create `PROJECT_ROOT/.kit/` as an empty placeholder. Nothing is copied yet — the board decides what gets copied in Step 6.

```bash
mkdir "PROJECT_ROOT\.kit"
```

**Rule:** Never modify any file under `KIT_PATH`. The kit is read-only source. The project owns everything under `.kit/`.

### 5c — Write CLAUDE.md

Write `PROJECT_ROOT/.claude/CLAUDE.md` using this exact template. Skills and agents are referenced by local `.kit/` paths — no `@` imports, no kit paths.

```markdown
# [Project Name]

## Active Task
Tasks: `.spec/tasks/` · Current: `.spec/tasks/task-001.md`

## Agents
`.kit/agents/` — entry point: `@company-coo`

## Commands
`.kit/commands/` — key: `/verify` · `/task-handoff` · `/tdd` · `/code-review` · `/wrapup`

## Bug Log
`bug-log.md` — date · what broke · root cause · fix · files.
```

Skills are added by `@company-coo` after board assessment — each selected skill is copied to `.kit/skills/` and listed here as a plain file path (not an `@` import).

### 5d — Write project-config.md

Write `PROJECT_ROOT/.claude/project-config.md` with a stub that `@company-coo` will populate:

```markdown
# Project Configuration

## Skill Library
- Path: KIT_PATH

## Selected Skills
_(To be populated by @company-coo)_

## Rules Active
- common
_(language-specific rules added by board after stack is decided)_

## Stack
_(To be decided and populated by the board)_
```

---

## Step 5f — Validate Generated Files

Before handing off to the board, verify each expected path exists.

**Real files (must exist):**
- `.claude/CLAUDE.md`
- `.claude/project-config.md`
- `.gitignore`
- `.git/` directory

**Copied directories (must exist under `.kit/`):**
- `.kit/agents/`
- `.kit/commands/`
- `.kit/hooks/`
- `.kit/contexts/`
- `.kit/rules/`

List any missing directories as warnings. Only proceed to Step 6 once all checks pass (or warnings are acknowledged by the user).

---

## Step 6 — Route to the Board and Trigger Planning

**This is the last action the skill takes.** Do not plan, select skills, or execute anything beyond this point.

### 6a — Classify the project with route-agents

Invoke the Skill tool with `route-agents`. Pass it the project description from Step 1. `route-agents` will return: the correct board entry point, the lead company, and supporting companies (if any).

### 6b — Hand off to company-coo with a complete brief

Invoke `@company-coo` with:
1. **Project description** — what is being built, for whom, and why (from Step 1)
2. **Infrastructure status** — git initialized, `.claude/CLAUDE.md` stub ready, `.kit/` folder created (empty, awaiting board content selection), validation passed (Step 5f)
3. **route-agents result** — lead company and supporting companies identified in Step 6a
4. **Instruction to company-coo:** "Select the lead CEO. Return the exact list of skills, agents, commands, hooks, contexts, and rules to copy from `KIT_PATH` into `.kit/`. The lead CEO must then invoke their company planning skill."

For anything `route-agents` cannot classify, default to `@company-coo` — the COO will reframe and route.

### 6c — Copy selected content into .kit/

Once `@company-coo` returns the list of selected skills, agents, commands, hooks, contexts, and rules — copy **only those items** from `KIT_PATH` into `PROJECT_ROOT/.kit/`:

```bash
# For each item the board selected:
xcopy /E /I /Y "KIT_PATH\skills\<category>\<skill>"   "PROJECT_ROOT\.kit\skills\<category>\<skill>"
xcopy /E /I /Y "KIT_PATH\agents\<company>\<agent>.md" "PROJECT_ROOT\.kit\agents\<company>\<agent>.md"
xcopy /E /I /Y "KIT_PATH\commands\<category>\<cmd>.md" "PROJECT_ROOT\.kit\commands\<category>\<cmd>.md"
xcopy /E /I /Y "KIT_PATH\rules\<ruleset>"             "PROJECT_ROOT\.kit\rules\<ruleset>"
# hooks and contexts are always included in full:
xcopy /E /I /Y "KIT_PATH\hooks"    "PROJECT_ROOT\.kit\hooks"
xcopy /E /I /Y "KIT_PATH\contexts" "PROJECT_ROOT\.kit\contexts"
```

**Rule:** Only copy what the board explicitly listed. Do not copy entire directories speculatively.

### 6d — Lead CEO runs the company planning skill

The lead CEO **must immediately** invoke their company's planning skill:

| Lead company | Planning skill to invoke |
|---|---|
| software-company | `planning-specification-architecture-software` |
| marketing-company | `planning-specification-architecture-marketing` |
| media-company | `planning-specification-architecture-media` |

The planning skill runs its three gated phases (brief/requirements → design → tasks). Only after all phases are user-approved does execution begin.

### 6e — Lead CEO writes skills into CLAUDE.md

After planning is complete and skills are confirmed, the lead CEO writes the final `## Skills` section into `PROJECT_ROOT/.claude/CLAUDE.md`. Replace the placeholder line with one plain file path per selected skill, pointing into `.kit/`:

```markdown
## Skills
.kit/skills/development/code-writing-software-development/SKILL.md
.kit/skills/core/strategic-compact/SKILL.md
.kit/rules/common/token-cost.md
[one line per additional skill the board selected]
```

Plain paths only — no `@` imports, no kit paths, no absolute paths outside the project.

**Rule:** Never skip `route-agents`. Never invoke a company CEO directly — always go through `company-coo`. Never let the board or COO create plans — planning belongs to the lead CEO's planning skill.

## Step 10 — Update / Regenerate Existing Projects

If re-running on an existing project:

1. Read existing `.claude/project-config.md` and `.claude/CLAUDE.md` first.
2. Read `KIT_PATH` from `project-config.md` — do not ask the user again.
3. Present the current list of `@`-imported skills from CLAUDE.md and ask the user if any should be added or removed.
4. For **new skills**: copy `KIT_PATH/skills/<category>/<skill>/` into `.kit/skills/<category>/<skill>/` and record the path in `project-config.md`. Add a plain file reference in `.claude/CLAUDE.md` under `## Skills`.
5. For **removed skills**: delete the skill folder from `.kit/skills/`, remove the reference from `.claude/CLAUDE.md`, and remove it from `project-config.md`.
6. Check that `.kit/` subdirectories all exist. Re-copy any that are missing.
7. Do NOT re-copy directories that already exist and are intact.
8. Ask the user before overwriting `CLAUDE.md` content other than the `@` import lines.

### 10a — Re-run Validation and Board Routing

After updating skills and CLAUDE.md, re-run Steps 5f and 6:

1. Read all existing `.spec/` files (`requirements.md`, `design.md`, `tasks/`) and pass them to `company-coo` as context, so the planning skill treats this as a revision, not a blank slate.
2. Validate `.kit/` directories (Step 5f) — re-copy any missing subdirectories from the kit.
3. Route the updated project description to `@company-coo` (Step 6) with a note that `.spec/` artifacts already exist and should be revised rather than replaced.
4. The lead CEO will invoke their company planning skill, which will revise the existing documents through its gated phases.

Preserve tasks already marked done — pass that context to the board so the planning skill only adds or revises incomplete tasks.

## Rules

### Hierarchy (CRITICAL)
- **ALWAYS route through the proper chain: board → company CEO → specialist.**
- **NEVER let board members (company-coo, chief-of-staff, chief-design-officer, people-operations-expert) create plans.** Plans originate from specialists delegated by company CEOs.
- **NEVER create a plan at routing level.** Analysis and infrastructure setup only — routing hands off to the board for delegation.
- **If you find yourself about to plan, stop.** The current agent is either at the wrong hierarchy level, or needs to route to a specialist.

### File Management
- **NEVER write any project file into the skill library directory.** Always confirm `PROJECT_ROOT` in Step 1b before writing anything.
- **If the CWD contains `.claude/skills/*/SKILL.md` files, it is the skill library — ask for a separate project folder.**
- Never include a skill the project doesn't need. Select only the 2–6 skills whose guidance applies to every task in this project.
- Never modify files under `KIT_PATH`. Copy into `.kit/` only — the kit is read-only source.
- Keep each section ≤ 3 lines. Cut every word that doesn't change behavior.
- If the project already has a `plan.md`, read it before generating — use it to pre-fill context for `requirements.md` and `design.md`.
- NEVER write `requirements.md`, `design.md`, or `tasks.md`. These are produced by the company planning skill after the board routes in Step 6.
- If the user's project has existing `workflows/` or `tools/`, mention them in the Layout section rather than overwriting.
- Always store project config in `.claude/project-config.md` — never hardcode deployment details in CLAUDE.md.
- If a source directory does not exist in the kit during copy, skip it silently and note it as missing in Step 5f validation output rather than failing.
- Never create `.claude/skills/` as a directory in the project. Skills are referenced by absolute `@` path only.
