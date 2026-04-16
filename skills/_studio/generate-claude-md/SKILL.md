---
name: generate-claude-md
description: Generate a pair of CLAUDE.md files for any project — one for planning phase, one for execution phase. Selects only the relevant skills based on what is being built. Both files are token-minimal. Use when starting a new project or restructuring an existing one.
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

## Step 3 — Analyze Project Description (STOP — No Infrastructure)

From Step 1, understand what's being built:
- Type of project (web app, CLI, data pipeline, etc.)
- Primary purpose and users
- Any existing codebase or starting point

**STOP HERE.** Do not set up infrastructure, create files, or select skills. Analysis only.

This becomes the input to the board in Step 6.

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
Also create `.claude/skills/` — this is where selected skill files will be copied using the correct folder structure.
Create `.spec/` in the project root if it doesn't exist. All planning artifacts (`plan.md`, `requirements.md`, `design.md`, `tasks.md`) go here.

## Step 5 — Create Junctions + Write CLAUDE.md

Projects reference the skill library directly — **no files are copied**. Directory junctions (Windows: `mklink /J`, zero disk usage) make agents, commands, rules, hooks, and contexts available. Skills are loaded into context only via explicit absolute-path `@` imports in `CLAUDE.md`.

### 5a — Determine Kit Path

Read the absolute path of the skill library. If unknown, ask the user once:
> "What is the absolute path to your claude_kit directory? (e.g. `C:/Users/Hp/Desktop/Experiment/claude_kit`)"

Store this as `KIT_PATH`. Use forward slashes throughout.

### 5b — Create Directory Junctions

Run these commands via Bash. Create `.claude/` and required subdirectories first if they don't exist.

```bash
# Agents and commands — entire kit dirs, on-demand only (zero context cost)
cmd /c mklink /J "PROJECT_ROOT\.claude\agents"   "KIT_PATH\agents"
cmd /c mklink /J "PROJECT_ROOT\.claude\commands" "KIT_PATH\commands"

# Hooks and contexts
cmd /c mklink /J "PROJECT_ROOT\.claude\hooks"    "KIT_PATH\hooks"
cmd /c mklink /J "PROJECT_ROOT\.claude\contexts" "KIT_PATH\contexts"

# Rules — always include common; add language-specific only if relevant
cmd /c mklink /J "PROJECT_ROOT\.claude\rules\common" "KIT_PATH\rules\common"
```

Language-specific rules junctions (add only if selected skill matches):

| Selected skill | Additional junction |
|---|---|
| `golang-patterns` | `cmd /c mklink /J "PROJECT\.claude\rules\golang" "KIT\rules\golang"` |
| `python-patterns` | `cmd /c mklink /J "PROJECT\.claude\rules\python" "KIT\rules\python"` |
| `kotlin-patterns` | `cmd /c mklink /J "PROJECT\.claude\rules\kotlin" "KIT\rules\kotlin"` |
| `build-website-web-app` or `java-patterns` | `cmd /c mklink /J "PROJECT\.claude\rules\typescript" "KIT\rules\typescript"` |
| `swift-patterns` | `cmd /c mklink /J "PROJECT\.claude\rules\swift" "KIT\rules\swift"` |

**If a junction target directory does not exist in the kit, skip it silently.**

### 5c — Write CLAUDE.md with Absolute @imports

Write `PROJECT_ROOT/.claude/CLAUDE.md` with absolute-path `@` imports for **only** the skills selected in Step 3. Do not include all skills — only the ones relevant to this project.

Always-include skills for every project (add their absolute paths):
- `code-writing-software-development` (`skills/development/code-writing-software-development/SKILL.md`)
- `strategic-compact` (`skills/core/strategic-compact/SKILL.md`)
- `token-cost` (`rules/common/token-cost.md`) — load via `@KIT_PATH/rules/common/token-cost.md`

Add only when the project needs them (do NOT include by default — each adds context cost):
- `continuous-learning` — long-running projects with evolving patterns
- `autonomous-agents-task-automation` — projects that orchestrate agents or automate pipelines
- `notebooklm` — projects that use NotebookLM as a second brain
- `wrapup` — projects with multi-session workflows

Add from Step 3 selections (examples):
- `tdd-workflow` → `skills/testing-quality/tdd-workflow/SKILL.md`
- `build-website-web-app` → `skills/development/build-website-web-app/SKILL.md`
- `security-review` → `skills/testing-quality/security-review/SKILL.md`
- `golang-patterns` → `skills/languages/golang-patterns/SKILL.md`

**@import format** (absolute path, forward slashes):
```
@KIT_PATH/skills/<category>/<skill-name>/SKILL.md
```

Template CLAUDE.md (skills will be added by `@company-coo` after assessing the project):
```markdown
# [Project Name]

## Skills
_(To be populated by @company-coo. The board will add only the skills needed for this specific project.)_

## Second Brain — NotebookLM
**ALWAYS check the AI Brain notebook BEFORE searching the codebase.**
When you need context about past decisions, patterns, architecture, or session history:
1. Read Brain notebook ID from memory files (`reference_brain_notebook.md`)
2. Query: `notebooklm ask "your question" --notebook <BRAIN_NOTEBOOK_ID>`
3. If the Brain returns relevant context → use it directly, DO NOT search the codebase
4. Only if the Brain has no match → fall back to codebase search, memory files, and .spec/ artifacts

## Active Agent Team

**Always start from the board. Describe your task naturally — the board will route internally.**

### Board (entry points)
| Agent | When to use |
|---|---|
| `@company-coo` | Building something, launching a product, cross-company initiatives, anything spanning software + marketing + media |
| `@chief-design-officer` | Design work that crosses company lines — software UI, marketing brand, and media visuals must be coherent together |
| `@people-operations-expert` | Hiring, comp bands, performance reviews, employment contracts, handbook, onboarding |
| `@chief-of-staff` | Comms triage (email, Slack, LINE, Messenger), decision tracking, weekly reviews, escalations between CEOs |

### How the board routes (ALWAYS respect hierarchy)
- `@company-coo` routes to: `@software-cto` (code, AI, infra, product, security, OS), `@chief-marketing-officer` (SEO, ads, growth, brand), `@chief-content-officer` (blog, video, podcast, social)
- Each CEO routes to specialists *within their company* for detailed planning and execution
- `@chief-design-officer` coordinates: `ui-design-expert` (inside software-company), `brand-expert` (inside marketing-company), `presentation-expert` (inside media-company)
- **CRITICAL:** Never reach past a board member into operating companies. Never reach past a CEO into specialists. Always respect the chain: board → CEO → specialist.
- **Planning rule:** Plans (requirements → design → tasks) originate from specialists, never from board members or CEOs directly.

## Advisor Strategy

Use a cost-efficient executor paired with Opus-level reasoning only on hard decisions.

```python
import anthropic
client = anthropic.Anthropic()
response = client.messages.create(
    model="claude-sonnet-4-6",        # executor drives the task
    tools=[
        {
            "type": "advisor_20260301",
            "name": "advisor",
            "model": "claude-opus-4-6",   # consulted only when needed
            "max_uses": 3,
        },
        # ...your other tools
    ],
    messages=[{"role": "user", "content": "..."}],
    extra_headers={"anthropic-beta": "advisor-tool-2026-03-01"},
)
```

**Rules:** Executor owns the work end-to-end. Advisor returns guidance only (400–700 tokens) — never writes files, calls tools, or produces user-facing output. Set `max_uses` to 2–4; >5 signals the task scope is too broad. Advisor tokens billed at Opus rates; executor at Sonnet/Haiku rates — overall cost stays below full-Opus runs.

## Active Feature
Feature: _(to be updated after planning starts)_
Tasks: .spec/tasks/
Current task: .spec/tasks/task-001.md
Branch: main

## Start Here
1. Read `## Active Feature` above — note the current task path.
2. Open the current task file — it is self-contained.
3. Skills section lists all rules you must follow — read them at session start.
4. Implement. Run `/task-handoff` when done.
5. Run `/wrapup` at end of session to save context to the AI Brain.

## Reference (load on demand — do not read at session start)
- Agents: `.claude/agents/` — invoke with `@agent-name`; use only when task specifies
- Commands: `.claude/commands/` — key ones: `/verify`, `/task-handoff`, `/save-session`, `/tdd`, `/code-review`, `/wrapup`
- Config: `.claude/project-config.md` — deployment targets, env vars, hosting
- Rules: `.claude/rules/` — applied automatically

## Bug Log
Append to `bug-log.md` immediately after any fix:
`## [YYYY-MM-DD] title | What broke: … | Root cause: … | Fix: … | File(s): …`

## Self-Check (before marking task done)
1. Acceptance Criteria in current task file — all pass?
2. Hardcoded values that should be env vars?
3. Upstream/downstream breakage?
4. `bug-log.md` updated if errors occurred?

## End of Session
Run `/wrapup` after `/task-handoff` or when ending any work session. This captures decisions, learnings, and open threads into the AI Brain notebook for future retrieval.

## Token Cost & Usage Limits

| Task | Model | Rule |
|---|---|---|
| Scaffolding, simple edits | Haiku 4.5 | Default for worker agents |
| Feature dev, multi-file | Sonnet 4.6 | Default executor |
| Architecture, hard decisions | Opus 4.6 (advisor only) | `max_uses: 2–4`, never full-session |

- Load only the skills this task needs — do not `@`-import unused skills.
- Read files with `Grep`/targeted `Read`; never `Read` an entire large file.
- Run `/compact` at phase boundaries (not mid-task) when tool call count exceeds 40.
- Set `max_tokens` explicitly on API calls with predictable output size.
- On `429` rate-limit: exponential back-off (1 s → 2 s → 4 s … cap 60 s).
- Full rules: `rules/common/token-cost.md`

## Output Discipline
Lead with the action. No preamble, no post-summary. Bullet points over prose.
```

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

### 5e — Write `.github/copilot-instructions.md`

Generate `PROJECT_ROOT/.github/copilot-instructions.md` as a Copilot-compatible mirror of `.claude/CLAUDE.md`. GitHub Copilot Chat auto-loads this file for every request in the repository.

**Transform the `@` import lines** — replace all `@KIT_PATH/skills/.../SKILL.md` lines with a single "Skills" section listing the same paths explicitly:

```markdown
## Skills — Read These Files for Coding Standards
When implementing tasks, read these files for detailed coding standards:
- KIT_PATH/skills/development/code-writing-software-development/SKILL.md
- KIT_PATH/skills/core/strategic-compact/SKILL.md
- KIT_PATH/rules/common/token-cost.md
- KIT_PATH/skills/core/wrapup/SKILL.md
- [one line per project-specific selected skill]
```

**Add `## Second Brain — NotebookLM` section at the top, before `## Start Here`:**

```markdown
## Second Brain — NotebookLM
**ALWAYS check the AI Brain BEFORE searching the codebase.**
When you need context about past decisions, patterns, architecture, or session history:
1. Read the Brain notebook ID from `.claude/memory/` files (look for `reference_brain_notebook.md`)
2. If the NotebookLM CLI is available, query: `notebooklm ask "your question" --notebook <BRAIN_NOTEBOOK_ID>`
3. Also check memory files in `.claude/memory/` and planning artifacts in `.spec/`
4. If the Brain or memory files return relevant context → use it directly, DO NOT search the codebase
5. Only if prior sources have no match → fall back to codebase search
This minimizes token usage by leveraging pre-summarized knowledge from prior sessions.
```

**Keep all other CLAUDE.md sections unchanged:** `## Active Feature`, `## Reference`, `## Bug Log`, `## Self-Check`, `## End of Session`, `## Output Discipline`.

**Update `## Start Here` item 3** from:
`"Skills are already loaded via @imports above — no need to load them manually."`
to:
`"Read the skill files listed in ## Skills above for coding standards."`

**Add a `## Key Commands` section** after `## Reference` — this is critical for GitHub Copilot, which cannot traverse directory junctions. When the user says "run /command-name", use `read_file` on the corresponding path below and follow its instructions exactly:

```markdown
## Key Commands
When the user types `/command-name`, read the corresponding file and follow its instructions exactly.

### Core Workflow
- `/checkpoint` → KIT_PATH/commands/core/checkpoint.md
- `/save-session` → KIT_PATH/commands/core/save-session.md
- `/resume-session` → KIT_PATH/commands/core/resume-session.md
- `/task-handoff` → KIT_PATH/commands/core/task-handoff.md
- `/sessions` → KIT_PATH/commands/core/sessions.md
- `/aside` → KIT_PATH/commands/core/aside.md
- `/projects` → KIT_PATH/commands/core/projects.md
- `/learn` → KIT_PATH/commands/core/learn.md
- `/learn-eval` → KIT_PATH/commands/core/learn-eval.md
- `/skill-create` → KIT_PATH/commands/core/skill-create.md
- `/evolve` → KIT_PATH/commands/core/evolve.md
- `/eval` → KIT_PATH/commands/core/eval.md
- `/promote` → KIT_PATH/commands/core/promote.md
- `/setup-pm` → KIT_PATH/commands/core/setup-pm.md
- `/harness-audit` → KIT_PATH/commands/core/harness-audit.md
- `/instinct-export` → KIT_PATH/commands/core/instinct-export.md
- `/instinct-import` → KIT_PATH/commands/core/instinct-import.md
- `/instinct-status` → KIT_PATH/commands/core/instinct-status.md
- `/wrapup` → KIT_PATH/skills/core/wrapup/SKILL.md

### Development
- `/verify` → KIT_PATH/commands/development/verify.md
- `/code-review` → KIT_PATH/commands/development/code-review.md
- `/build-fix` → KIT_PATH/commands/development/build-fix.md
- `/refactor-clean` → KIT_PATH/commands/development/refactor-clean.md
- `/update-docs` → KIT_PATH/commands/development/update-docs.md
- `/update-codemaps` → KIT_PATH/commands/development/update-codemaps.md
- `/pm2` → KIT_PATH/commands/development/pm2.md
- `/prompt-optimize` → KIT_PATH/commands/development/prompt-optimize.md

### Testing & Quality
- `/tdd` → KIT_PATH/commands/testing-quality/tdd.md
- `/e2e` → KIT_PATH/commands/testing-quality/e2e.md
- `/quality-gate` → KIT_PATH/commands/testing-quality/quality-gate.md
- `/test-coverage` → KIT_PATH/commands/testing-quality/test-coverage.md

### Planning
- `/plan` → KIT_PATH/commands/planning/plan.md
- `/orchestrate` → KIT_PATH/commands/planning/orchestrate.md
- `/model-route` → KIT_PATH/commands/planning/model-route.md
- `/loop-start` → KIT_PATH/commands/planning/loop-start.md
- `/loop-status` → KIT_PATH/commands/planning/loop-status.md
- `/multi-plan` → KIT_PATH/commands/planning/multi-plan.md
- `/multi-execute` → KIT_PATH/commands/planning/multi-execute.md
- `/multi-workflow` → KIT_PATH/commands/planning/multi-workflow.md
- `/multi-backend` → KIT_PATH/commands/planning/multi-backend.md
- `/multi-frontend` → KIT_PATH/commands/planning/multi-frontend.md

### Languages
- `/go-build` → KIT_PATH/commands/languages/go-build.md
- `/go-review` → KIT_PATH/commands/languages/go-review.md
- `/go-test` → KIT_PATH/commands/languages/go-test.md
- `/gradle-build` → KIT_PATH/commands/languages/gradle-build.md
- `/kotlin-build` → KIT_PATH/commands/languages/kotlin-build.md
- `/kotlin-review` → KIT_PATH/commands/languages/kotlin-review.md
- `/kotlin-test` → KIT_PATH/commands/languages/kotlin-test.md
- `/python-review` → KIT_PATH/commands/languages/python-review.md

### Specialized
- `/claw` → KIT_PATH/commands/specialized/claw.md
```

**Rule:** Skill paths in `copilot-instructions.md` must match exactly the `@` imports in `.claude/CLAUDE.md`. Both files always list the same skill set.

---

## Step 5f — Validate Generated Files

Before handing off to the board, verify each expected path exists.

**Real files (must exist):**
- `.claude/CLAUDE.md`
- `.claude/project-config.md`
- `.gitignore`
- `.github/copilot-instructions.md`
- `.git/` directory

**Junctions (must resolve correctly — check a known file through each):**
- `.claude/agents/` → `KIT_PATH/agents/`
- `.claude/commands/` → `KIT_PATH/commands/`
- `.claude/hooks/` → `KIT_PATH/hooks/`
- `.claude/contexts/` → `KIT_PATH/contexts/`
- `.claude/rules/common/` → `KIT_PATH/rules/common/`

List any missing files or broken junctions as warnings. Only proceed to Step 6 once all checks pass (or warnings are acknowledged by the user).

---

## Step 6 — Route to the Board and Trigger Planning

**This is the last action the skill takes.** Do not plan, select skills, or execute anything beyond this point.

### 6a — Classify the project with route-agents

Invoke the Skill tool with `route-agents`. Pass it the project description from Step 1. `route-agents` will return: the correct board entry point, the lead company, and supporting companies (if any).

### 6b — Hand off to company-coo with a complete brief

Invoke `@company-coo` with:
1. **Project description** — what is being built, for whom, and why (from Step 1)
2. **Infrastructure status** — git initialized, `.claude/CLAUDE.md` stub ready (Skills section empty, awaits board decision), junctions created, validation passed (Step 5f)
3. **route-agents result** — lead company and supporting companies identified in Step 6a
4. **Instruction to company-coo:** "Select the lead CEO based on the route-agents result. Route to that CEO only — do not route to specialists directly. The lead CEO must invoke their company planning skill before any execution begins."

For anything `route-agents` cannot classify, default to `@company-coo` — the COO will reframe and route.

### 6c — Lead CEO runs the company planning skill

`company-coo` routes to the lead CEO. The lead CEO **must immediately** invoke their company's planning skill:

| Lead company | Planning skill to invoke |
|---|---|
| software-company | `planning-specification-architecture-software` |
| marketing-company | `planning-specification-architecture-marketing` |
| media-company | `planning-specification-architecture-media` |

The planning skill runs its three gated phases (brief/requirements → design → tasks). Only after all phases are user-approved does execution begin.

**Rule:** Never skip `route-agents`. Never invoke a company CEO directly — always go through `company-coo`. Never let the board or COO create plans — planning belongs to the lead CEO's planning skill.

## Step 10 — Update / Regenerate Existing Projects

If re-running on an existing project:

1. Read existing `.claude/project-config.md` and `.claude/CLAUDE.md` first.
2. Read `KIT_PATH` from `project-config.md` — do not ask the user again.
3. Present the current list of `@`-imported skills from CLAUDE.md and ask the user if any should be added or removed.
4. For **new skills**: add a new `@KIT_PATH/skills/<category>/<skill>/SKILL.md` line to `.claude/CLAUDE.md`, add the skill to `project-config.md`, and add the corresponding absolute path line to the `## Skills` section of `.github/copilot-instructions.md`.
5. For **removed skills**: delete the corresponding `@` line from `.claude/CLAUDE.md`, remove from `project-config.md`, and remove the matching line from `.github/copilot-instructions.md`. Both files must always list the same skill set.
6. Check that all junctions exist and resolve correctly. Create any missing junctions (e.g. if a new language ruleset is needed).
7. Do NOT recreate junctions that already exist and resolve correctly.
8. Ask the user before overwriting `CLAUDE.md` content other than the `@` import lines.

### 10a — Re-run Validation and Board Routing

After updating skills and CLAUDE.md, re-run Steps 5f and 6:

1. Read all existing `.spec/` files (`requirements.md`, `design.md`, `tasks/`) and pass them to `company-coo` as context, so the planning skill treats this as a revision, not a blank slate.
2. Validate all junctions and `@imports` (Step 5f) — create any missing junctions for newly added language rulesets.
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
- Never copy skill content into the project — use absolute `@KIT_PATH/skills/.../SKILL.md` imports in `.claude/CLAUDE.md`.
- Keep each section ≤ 3 lines. Cut every word that doesn't change behavior.
- If the project already has a `plan.md`, read it before generating — use it to pre-fill context for `requirements.md` and `design.md`.
- NEVER write `requirements.md`, `design.md`, or `tasks.md`. These are produced by the company planning skill after the board routes in Step 6.
- If the user's project has existing `workflows/` or `tools/`, mention them in the Layout section rather than overwriting.
- Always store project config in `.claude/project-config.md` — never hardcode deployment details in CLAUDE.md.
- If a junction target directory does not exist in the kit, skip it silently and note it as missing in Step 5f validation output rather than failing.
- Never create `.claude/skills/` as a directory in the project. Skills are referenced by absolute `@` path only.
