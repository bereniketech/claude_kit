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

## Step 2 — Gather Deployment & Hosting Info (Smart Defaults)

After understanding the project type, infer sensible defaults rather than asking 8 questions upfront.

### Default Profiles by Project Type

| Project Type | Package Manager | Hosting | Database | Auth |
|---|---|---|---|---|
| Web app (React/Next.js) | bun | Vercel | Supabase | Supabase Auth |
| Landing page / static site | bun | Vercel | N/A | N/A |
| CLI tool / library | npm | N/A | N/A | N/A |
| API / backend service | npm | Railway | Supabase | JWT |
| Data pipeline | pip/uv | N/A | N/A | N/A |
| Mobile app (React Native) | bun | N/A | Supabase | Supabase Auth |

### Interaction

1. Pre-fill all fields using the matching default profile.
2. Present the pre-filled config to the user in one block.
3. Ask: **"Here are the defaults for your project type. Change anything that's wrong, or say 'looks good' to continue."**
4. Apply any overrides the user provides.

Only ask follow-up questions for fields the user explicitly marks as needing clarification.

### Store in `.claude/project-config.md`

```markdown
# Project Configuration

## Skill Library
- Path: [absolute path to the skill library — store so future runs auto-detect]

## GitHub
- Repo: [url or "not yet created"]
- Branch: main
- Visibility: [public/private]

## Hosting
- Provider: [provider]
- Domain: [domain or "none"]

## CI/CD
- Pipeline: [tool or "none"]

## Environment Variables
[List of required env var names — NO values]

## Database
- Provider: [provider or "none"]

## Auth
- Provider: [provider or "none"]

## Package Manager
- [npm/pnpm/yarn/bun]
```

If any field is unknown or not applicable, write "TBD" or "N/A". This file is referenced during execution for deployment tasks.

## Step 3 — Select Relevant Skills

### Core / Meta
| If the project involves… | Include skill |
|---|---|
| Generating CLAUDE.md for a project | `generate-claude-md` |
| Planning new features onto an existing project | `new-features-updates` |
| Auditing the skill library itself | `skill-stocktake` |
| Installing ECC skills/rules | `configure-ecc` |
| Extracting patterns from sessions | `continuous-learning` |
| Context window compaction | `strategic-compact` |
| Eval-driven AI development | `eval-harness` |
| End-of-session knowledge capture | `wrapup` |

### Development
| If the project involves… | Include skill |
|---|---|
| Any code writing / refactoring | `code-writing-software-development` |
| Web app / landing page / UI | `build-website-web-app` |
| IDE pair programming assistance | `ide-integration-pair-programming` |
| REST API design | `api-design` |
| Content caching patterns | `content-hash-cache-pattern` |
| Bug hunting / debugging methodology | `systematic-debugging` |
| Branch merge / PR / cleanup workflow | `branch-completion` |

### Planning & Architecture
| If the project involves… | Include skill |
|---|---|
| Requirements / architecture | `planning-specification-architecture` |
| Autonomous multi-step tasks | `autonomous-agents-task-automation` |
| Multiple AI models / providers | `multi-model-model-agnostic-platforms` |
| Design ideation / brainstorming | `brainstorming` |
| Parallel agent orchestration | `parallel-agent-dispatch` |

### Testing & Quality
| If the project involves… | Include skill |
|---|---|
| Test-driven development | `tdd-workflow` |
| Security audit / OWASP review | `security-review` |
| Automated security scanning | `security-scan` |
| Playwright / E2E testing | `e2e-testing` |

### Data & Backend
| If the project involves… | Include skill |
|---|---|
| PostgreSQL / Supabase DB | `postgres-patterns` |
| Database schema migrations | `database-migrations` |
| ClickHouse / OLAP analytics | `clickhouse-io` |

### Languages
| If the project involves… | Include skill |
|---|---|
| Go / Golang projects | `golang-patterns` |
| Python / Django projects | `python-patterns` |
| Kotlin / Ktor projects | `kotlin-patterns` |
| Java / Spring Boot projects | `java-patterns` |
| Swift / SwiftUI / iOS projects | `swift-patterns` |
| C++ projects | `cpp-patterns` |
| Android / KMP projects | `android-patterns` |
| Perl projects | `perl-patterns` |

### Claude / AI Platform
| If the project involves… | Include skill |
|---|---|
| Claude API / Anthropic SDK | `claude-developer-platform` |
| NotebookLM / podcast generation | `notebooklm` |

### Research & Docs
| If the project involves… | Include skill |
|---|---|
| Web search / knowledge retrieval | `research-information-retreival` |
| Writing docs / READMEs / reports | `document-content-writing-editing` |
| Notes / knowledge bases / tasks | `knowledge-management-productivity` |

### DevOps & Infrastructure
| If the project involves… | Include skill |
|---|---|
| Shell scripts / CI/CD / DevOps | `terminal-cli-devops` |
| Git worktree-based isolation | `git-worktrees` |

### UI & Design
| If the project involves… | Include skill |
|---|---|
| UI components / design system | `presentations-ui-design` |
| UI/UX design decisions / style selection | `ui-ux-pro-max` |
| Logo, CIP, icon, or banner design | `design` |
| Brand identity / voice / guidelines | `brand` |
| Design tokens / component specs | `design-system` |
| Banner / social media assets | `banner-design` |
| HTML presentations / pitch decks | `slides` |
| shadcn/ui / Tailwind styling | `ui-styling` |

### Integrations
| If the project involves… | Include skill |
|---|---|
| Twitter/X API integration | `x-api` |
| AI image/video generation | `fal-ai-media` |
| Video editing pipelines | `video-editing` |
| Video database / streaming | `videodb` |
| Document OCR / PDF processing | `nutrient-document-processing` |
| Visa / translation documents | `visa-doc-translate` |
| WhatsApp messaging automation | `whatsapp-automation` |
| GitHub Copilot autonomous task loop | `copilot-autonomous-tasks` |

### Domain
| If the project involves… | Include skill |
|---|---|
| Logistics / supply chain | `logistics-supply-chain` |
| Trade / customs compliance | `trade-compliance` |
| Energy procurement | `energy-procurement` |

### Specialized
| If the project involves… | Include skill |
|---|---|
| NanoClaw REPL sessions | `nanoclaw-repl` |
| Large RFC multi-agent features | `ralphinho-rfc-pipeline` |

Planning CLAUDE.md always includes `planning-specification-architecture`.
Execution CLAUDE.md always includes these eight skills regardless of project type — they are the foundation of structured AI-assisted development:

| Skill | What it enforces |
|---|---|
| `continuous-learning` | Persists patterns across sessions — prevents repeating debugging and re-explaining conventions |
| `strategic-compact` | Prevents context rot — compacts at phase boundaries so architectural decisions stay coherent |
| `tdd-workflow` | 80%+ test coverage enforced from the first line — Red/Green/Refactor on every feature |
| `code-writing-software-development` | Coding standards, read-before-edit discipline, and the 6-phase verification loop (build → type → lint → test → security → diff) |
| `security-review` | OWASP Top 10 checklist on every feature touching auth, input, secrets, or APIs — catches issues before code review |
| `autonomous-agents-task-automation` | Parallel execution, model routing, and subagent delegation — faster feature completion with less context switching |
| `notebooklm` | Second brain — every session's knowledge is queryable; "check Brain first" reduces token burn by leveraging pre-summarized context |
| `wrapup` | Session persistence — captures decisions, learnings, and open threads at session end and pushes them to the NotebookLM Brain |

Execution CLAUDE.md includes all other selected skills — never `planning-specification-architecture`.

## Step 4 — Initialize Git-Ready Project

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

If the project needs environment variables (from Step 2), create `.env.example` with the variable names and placeholder comments — no real values:

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
- `continuous-learning` (`skills/core/continuous-learning/SKILL.md`)
- `strategic-compact` (`skills/core/strategic-compact/SKILL.md`)
- `autonomous-agents-task-automation` (`skills/planning/autonomous-agents-task-automation/SKILL.md`)
- `notebooklm` (`skills/ai-platform/notebooklm/SKILL.md`)
- `wrapup` (`skills/core/wrapup/SKILL.md`)

Add from Step 3 selections (examples):
- `tdd-workflow` → `skills/testing-quality/tdd-workflow/SKILL.md`
- `build-website-web-app` → `skills/development/build-website-web-app/SKILL.md`
- `security-review` → `skills/testing-quality/security-review/SKILL.md`
- `golang-patterns` → `skills/languages/golang-patterns/SKILL.md`

**@import format** (absolute path, forward slashes):
```
@KIT_PATH/skills/<category>/<skill-name>/SKILL.md
```

Example CLAUDE.md (adjust skills per project):
```markdown
# [Project Name]

@C:/Users/Hp/Desktop/Experiment/claude_kit/skills/development/code-writing-software-development/SKILL.md
@C:/Users/Hp/Desktop/Experiment/claude_kit/skills/core/continuous-learning/SKILL.md
@C:/Users/Hp/Desktop/Experiment/claude_kit/skills/core/strategic-compact/SKILL.md
@C:/Users/Hp/Desktop/Experiment/claude_kit/skills/planning/autonomous-agents-task-automation/SKILL.md
@C:/Users/Hp/Desktop/Experiment/claude_kit/skills/testing-quality/tdd-workflow/SKILL.md
@C:/Users/Hp/Desktop/Experiment/claude_kit/skills/development/build-website-web-app/SKILL.md
@C:/Users/Hp/Desktop/Experiment/claude_kit/skills/ai-platform/notebooklm/SKILL.md
@C:/Users/Hp/Desktop/Experiment/claude_kit/skills/core/wrapup/SKILL.md

## Second Brain — NotebookLM
**ALWAYS check the AI Brain notebook BEFORE searching the codebase.**
When you need context about past decisions, patterns, architecture, or session history:
1. Read Brain notebook ID from memory files (`reference_brain_notebook.md`)
2. Query: `notebooklm ask "your question" --notebook <BRAIN_NOTEBOOK_ID>`
3. If the Brain returns relevant context → use it directly, DO NOT search the codebase
4. Only if the Brain has no match → fall back to codebase search, memory files, and .spec/ artifacts
This is mandatory. Skipping the Brain check wastes tokens re-reading files that have already been summarized.

## Active Feature
Feature: {feature-name or "initial build"}
Tasks: .spec/tasks/
Current task: .spec/tasks/task-001.md
Branch: main

## Start Here
1. Read `## Active Feature` above — note the current task path.
2. Open the current task file — it is self-contained.
3. Skills are already loaded via @imports above — no need to load them manually.
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

## Output Discipline
Lead with the action. No preamble, no post-summary. Bullet points over prose.
```

### 5d — Write project-config.md

Write `PROJECT_ROOT/.claude/project-config.md`:

```markdown
# Project Configuration

## Skill Library
- Path: KIT_PATH

## Selected Skills
- code-writing-software-development → skills/development/code-writing-software-development
- continuous-learning → skills/core/continuous-learning
- [list all @imported skills with their kit-relative path]

## Rules Active
- common
- [language-specific if applicable]

## GitHub
- Repo: [url or "not yet created"]
- Branch: main

## Hosting
- Provider: [provider]
- Domain: [domain or "none"]

## Database
- Provider: [provider or "none"]

## Package Manager
- [npm/pnpm/yarn/bun/pip]

## Environment Variables
[List of required env var names — NO values]
```

### 5e — Write `.github/copilot-instructions.md`

Generate `PROJECT_ROOT/.github/copilot-instructions.md` as a Copilot-compatible mirror of `.claude/CLAUDE.md`. GitHub Copilot Chat auto-loads this file for every request in the repository.

**Transform the `@` import lines** — replace all `@KIT_PATH/skills/.../SKILL.md` lines with a single "Skills" section listing the same paths explicitly:

```markdown
## Skills — Read These Files for Coding Standards
When implementing tasks, read these files for detailed coding standards:
- KIT_PATH/skills/development/code-writing-software-development/SKILL.md
- KIT_PATH/skills/core/continuous-learning/SKILL.md
- KIT_PATH/skills/core/strategic-compact/SKILL.md
- KIT_PATH/skills/planning/autonomous-agents-task-automation/SKILL.md
- KIT_PATH/skills/ai-platform/notebooklm/SKILL.md
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

## Step 6 — Generate Planning CLAUDE.md

Save as `.claude/CLAUDE.planning.md` (inside `.claude/`, not the project root).

Template (fill in `[...]` from context — keep every section to 1–3 lines):

```markdown
# Plan: [Project Name]

## Goal
[One sentence: what this project produces and for whom.]

## Constraints
[Bullet list: language/framework locks, API limits, deadlines, style rules. Omit if none.]

## Deliverables
The plan must produce:
- `.spec/plan.md` — high-level project overview: goal, tech stack, architecture diagram, file structure
- `.spec/requirements.md` — user stories and acceptance criteria (EARS format)
- `.spec/design.md` — architecture, data models, API design, ADRs, security, performance
- `.spec/tasks.md` — ordered task list with acceptance criteria per task

## Instructions
Use /planning-specification-architecture.
Write `plan.md` first as the high-level overview, then follow the skill's 3-phase gated workflow: requirements → user approves → design → user approves → tasks → user approves.
Do not write implementation code. Do not skip approval gates.
Save each artifact only after the user explicitly approves that phase.
```

## Step 7 — Run Planning Phase via /planning-specification-architecture

### 7a — Write plan.md directly
Write `.spec/plan.md` yourself — the high-level project overview: goal, constraints, tech stack rationale, architecture diagram (Mermaid), file/folder structure. This is the only `.spec/` file `generate-claude-md` writes directly.

### 7b — Invoke the planning skill
Call the Skill tool with `planning-specification-architecture` before writing any of the three gated files. Do NOT proceed without this invocation.

### 7c — Follow the 3-phase gated workflow
Using project context from Steps 1–2, follow the planning skill's gated workflow exactly:

**Phase 1 — Requirements**
- Draft the full content of `.spec/requirements.md` (user stories + EARS acceptance criteria for every major feature) and present it to the user as inline text — do NOT save the file yet.
- Ask: "Does the requirements document look right? If so, we can move on to the design."
- Wait for explicit approval (`yes`, `looks good`, `approved`, or equivalent).
- ONLY after approval: save `.spec/requirements.md`.

**Phase 2 — Design**
- Draft the full content of `.spec/design.md` (architecture Mermaid diagram, ERD, API design, ADRs, error handling, security, performance) and present it as inline text — do NOT save the file yet.
- Ask: "Does the design look good? If so, we can move on to the implementation plan."
- Wait for explicit approval.
- ONLY after approval: save `.spec/design.md`.

**Phase 3 — Tasks**
- Draft the full content of `.spec/tasks.md` (ordered tasks with `_Requirements:_`, `_Skills:_`, and `**AC:**`) and present it as inline text — do NOT save the file yet.
- Ask: "Do the tasks look good? Once approved, the spec is ready for implementation."
- Wait for explicit approval.
- ONLY after approval: save `.spec/tasks.md`.

All three files must reflect the deployment/hosting choices in `.claude/project-config.md`.

### HARD RULES — violations are prohibited
- **NEVER write `requirements.md`, `design.md`, or `tasks.md` without first invoking the Skill tool with `planning-specification-architecture`.**
- **NEVER save any of the three gated files before the user explicitly approves that phase.**
- **NEVER present and save in the same step — present first, wait, then save.**
- **NEVER combine phases — present and approve one phase at a time.**

### 7d — Run Task Enrichment

Immediately after `.spec/tasks.md` is saved (Phase 3 approved), create self-contained individual task files:

1. Create `.spec/tasks/` directory.
2. For each task in `tasks.md`, create `task-NNN.md` (zero-padded, e.g. `task-001.md`, `task-002.md`).
3. Populate `## Session Bootstrap` with only the skills that task requires (from its `_Skills:_` line) — not all project skills. Reference them by name only (e.g. "tdd-workflow") — the actual content loads via @imports in `.claude/CLAUDE.md`.
4. Populate Objective, Implementation Steps, and Acceptance Criteria from the corresponding entry in `tasks.md`.
5. For **greenfield projects** (no existing source files yet): write `[greenfield — no existing files to reference]` in the Key Patterns section and leave Key Code Snippets empty.
6. For **Step 10 (update/regenerate)**: use `Glob` and `Grep` on real `PROJECT_ROOT` source files. Embed actual code snippets (interfaces, base classes, key types) directly into the Key Code Snippets block — so the implementer reads zero files during execution.
7. Leave all Handoff sections with placeholders — `/task-handoff` fills these during execution.

**Rule:** Task Enrichment does NOT require a separate user approval gate. It is a mechanical transformation of already-approved `tasks.md` content.

## Step 8 — Output

After writing all files:
1. Print a one-line summary of which skills were selected and why (list their kit paths as they appear in `.claude/CLAUDE.md`).
2. Print the deployment/hosting config summary from `.claude/project-config.md`.
3. Tell the user: `.spec/plan.md`, `.spec/requirements.md`, `.spec/design.md`, and `.spec/tasks.md` are ready. Individual task files are in `.spec/tasks/` — start with `task-001.md`. Run `/task-handoff` after each task completes.
4. List the junctions created in `.claude/` so the user can see which agents, commands, rules, hooks, and contexts are available via the kit.
5. Remind the user: editing any skill in the kit takes effect on the next Claude Code session start. Editing agents/commands takes effect immediately.
6. Note: `.github/copilot-instructions.md` was created — it mirrors CLAUDE.md with explicit skill file paths. GitHub Copilot Chat auto-loads this file, so switching to Copilot and asking it to complete a task file will work out of the box.

## Step 9 — Validate Generated Files

After all files are written, verify each expected path exists. Check for:

**Real files (must exist as actual files):**
- `.claude/CLAUDE.md` — contains `@KIT_PATH/skills/.../SKILL.md` imports for each selected skill
- `.claude/CLAUDE.planning.md`
- `.claude/project-config.md` — contains kit path and selected skill list
- `.spec/plan.md`
- `.spec/requirements.md`
- `.spec/design.md`
- `.spec/tasks.md`
- `.spec/tasks/task-001.md`
- `.gitignore`
- `.env.example` (if env vars were defined)
- `.git/` directory (git repo initialized)
- `.github/copilot-instructions.md` — contains the same skills as `.claude/CLAUDE.md` as readable file paths

**Junctions (must exist as directory junctions pointing to the kit):**
- `.claude/agents/` → `KIT_PATH/agents/`
- `.claude/commands/` → `KIT_PATH/commands/`
- `.claude/hooks/` → `KIT_PATH/hooks/`
- `.claude/contexts/` → `KIT_PATH/contexts/`
- `.claude/rules/common/` → `KIT_PATH/rules/common/`
- Language-specific rules junctions (if applicable)

**Verify junctions resolve correctly** — check that a known file is accessible through each junction (e.g. `.claude/hooks/hooks.json` should be readable).

**Verify @imports in CLAUDE.md** — each `@KIT_PATH/skills/.../SKILL.md` path must point to a file that actually exists in the kit.

List any missing files or broken junctions as warnings. Do not declare success until all checks pass.

## Step 10 — Update / Regenerate Existing Projects

If re-running on an existing project:

1. Read existing `.claude/project-config.md`, `.claude/CLAUDE.md`, and `.claude/CLAUDE.planning.md` first.
2. Read `KIT_PATH` from `project-config.md` — do not ask the user again.
3. Present the current list of `@`-imported skills from CLAUDE.md and ask the user if any should be added or removed.
4. For **new skills**: add a new `@KIT_PATH/skills/<category>/<skill>/SKILL.md` line to `.claude/CLAUDE.md`, add the skill to `project-config.md`, and add the corresponding absolute path line to the `## Skills` section of `.github/copilot-instructions.md`.
5. For **removed skills**: delete the corresponding `@` line from `.claude/CLAUDE.md`, remove from `project-config.md`, and remove the matching line from `.github/copilot-instructions.md`. Both files must always list the same skill set.
6. Check that all junctions exist and resolve correctly. Create any missing junctions (e.g. if a new language ruleset is needed).
7. Do NOT recreate junctions that already exist and resolve correctly.
8. Ask the user before overwriting `CLAUDE.md` content other than the `@` import lines.

### 10a — Re-run the Full Planning Phase

After updating skills and CLAUDE.md, run the full planning phase exactly as in Step 7 — including the gated workflow. Treat it as a **revision**, not a blank slate:

1. Read all existing `.spec/` files (`plan.md`, `requirements.md`, `design.md`, `tasks.md`) before drafting anything. Use their current content as the baseline.
2. Update `.spec/plan.md` directly to reflect any new goals, stack changes, or scope discovered during the update.
3. Invoke the Skill tool with `planning-specification-architecture` before writing any of the three gated files.
4. Follow the same 3-phase gated workflow as Step 7c — present each updated draft inline, wait for explicit approval, then save:
   - **Phase 1 — Requirements:** present updated `requirements.md` showing what changed. Ask for approval before saving.
   - **Phase 2 — Design:** present updated `design.md` reflecting any architectural changes. Ask for approval before saving.
   - **Phase 3 — Tasks:** present updated `tasks.md` with new/revised tasks added and completed tasks preserved. Ask for approval before saving.

**HARD RULES (same as Step 7):**
- **NEVER write `requirements.md`, `design.md`, or `tasks.md` without first invoking the Skill tool with `planning-specification-architecture`.**
- **NEVER save any gated file before the user explicitly approves that phase.**
- **NEVER present and save in the same step.**
- **NEVER combine phases.**
- Preserve tasks already marked done — only add or revise incomplete tasks.

## Rules

- **NEVER write any project file into the skill library directory.** Always confirm `PROJECT_ROOT` in Step 1b before writing anything.
- **If the CWD contains `.claude/skills/*/SKILL.md` files, it is the skill library — ask for a separate project folder.**
- Never include a skill the project doesn't need. Select only the 2–6 skills whose guidance applies to every task in this project.
- Never copy skill content into the project — use absolute `@KIT_PATH/skills/.../SKILL.md` imports in `.claude/CLAUDE.md`.
- Keep each section ≤ 3 lines. Cut every word that doesn't change behavior.
- If the project already has a `plan.md`, read it before generating — use it to pre-fill context for `requirements.md` and `design.md`.
- NEVER write `requirements.md`, `design.md`, or `tasks.md` directly. These three files are ONLY produced through the gated workflow in Step 7 (new projects) or Step 10a (existing projects) after explicit user approval of each phase.
- If the user's project has existing `workflows/` or `tools/`, mention them in the Layout section rather than overwriting.
- Always store project config in `.claude/project-config.md` — never hardcode deployment details in CLAUDE.md.
- If a junction target directory does not exist in the kit, skip it silently and note it as missing in Step 8 output rather than failing.
- Never create `.claude/skills/` as a directory in the project. Skills are referenced by absolute `@` path only.
