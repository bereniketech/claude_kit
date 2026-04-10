---
name: new-features-updates
description: Plan and scaffold new features onto an existing project. Analyzes the current codebase, ingests specs (text or folder of docs/images), installs missing skill library components via @imports and junctions (zero-copy), creates a feature branch and spec folder, then delegates to planning-specification-architecture for the gated requirements → design → tasks workflow.
---

# New Features & Updates

Use this skill when new specifications arrive for an existing project. It bridges the gap between `generate-claude-md` (used once at project start) and active feature development.

---

## 1. Gather New Specifications

If the user has not already provided specs, ask exactly one question:

> "Describe the new feature or change — or give me a path to a folder with spec docs."

Derive `FEATURE_NAME` from the spec (short, lowercase, hyphenated, e.g. `user-auth`, `payment-flow`). Confirm with the user if unclear.

If a folder path is provided, scan it with `Glob` for `*.md`, `*.txt`, `*.pdf`, `*.png`, `*.jpg`, `*.jpeg`, `*.docx` and read all files. Combine with any text description provided.

---

## 2. Confirm Project Root

Check if CWD contains existing project files (`package.json`, `pyproject.toml`, `go.mod`, source folders).

If CWD is the skill library (has `.claude/skills/*/SKILL.md`), ask:
> "What is the path to your existing project?"

Set `PROJECT_ROOT`. Read `PROJECT_ROOT/.claude/project-config.md` to get `KIT_PATH` (the absolute path to the skill library) — needed in Step 4.

**Rule:** Never write any file outside `PROJECT_ROOT`. Never treat the skill library directory as the project.

---

## 3. Analyze Existing Codebase

Use `Glob` and `Grep` to understand what already exists in `PROJECT_ROOT`:

- Top-level folder structure
- Tech stack (infer from `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, etc.)
- Key source files: entry points, API routes, data models, main modules
- Existing `.spec/` folders and files — read all present to understand prior plans
- Already-installed components: read `.claude/CLAUDE.md` to list `@` imported skills
- Existing junctions: check `.claude/agents/`, `.claude/commands/`, `.claude/rules/`, `.claude/hooks/`, `.claude/contexts/`

**Rule:** Never skip this step. Planning without reading the existing code produces specs that conflict with what already exists.

### 3b — Conditional Architect Analysis

Only if the feature spec explicitly involves one or more of:
- A new service or module boundary
- A change to the existing data model
- Integration with a new external system

…then call the `@architect` agent with: the existing codebase summary from Step 3 + the feature spec from Step 1. Use its output to inform the spec structure in Step 6.

**Otherwise: skip. Do not call @architect for features that extend existing patterns.**

---

## 4. Check Skill Library for Missing Components

Read `KIT_PATH` from `PROJECT_ROOT/.claude/project-config.md`. All installs are done by adding `@` import lines to `.claude/CLAUDE.md` — **never copy files**.

### 4a — Baseline Mandatory Skills (always install if absent)

Regardless of what the new feature involves, every project must have all eight of these. Check `.claude/CLAUDE.md` for existing `@` imports and add silently if missing — no user confirmation needed:

| Skill | Kit-relative path | What it enforces |
|---|---|---|
| `code-writing-software-development` | `skills/development/code-writing-software-development/SKILL.md` | Coding standards, read-before-edit, 6-phase verification loop |
| `tdd-workflow` | `skills/testing-quality/tdd-workflow/SKILL.md` | 80%+ test coverage, Red/Green/Refactor, CI gate before every PR |
| `security-review` | `skills/testing-quality/security-review/SKILL.md` | OWASP Top 10 before any feature touching auth, input, secrets, or APIs |
| `autonomous-agents-task-automation` | `skills/planning/autonomous-agents-task-automation/SKILL.md` | Parallel execution, model routing, subagent delegation |
| `continuous-learning` | `skills/core/continuous-learning/SKILL.md` | Persists patterns across sessions — no repeated debugging |
| `strategic-compact` | `skills/core/strategic-compact/SKILL.md` | Context rot prevention — compacts at phase boundaries |
| `notebooklm` | `skills/ai-platform/notebooklm/SKILL.md` | Second brain — session knowledge is queryable |
| `wrapup` | `skills/core/wrapup/SKILL.md` | Session persistence — captures decisions and open threads at session end |

Add each missing skill as an `@` import line in `PROJECT_ROOT/.claude/CLAUDE.md`:
```
@KIT_PATH/skills/<category>/<skill-name>/SKILL.md
```

### 4b — Conditional Skills

Using the feature spec from Step 1, cross-check against already-imported skills in `.claude/CLAUDE.md`. Apply the same selection logic as `generate-claude-md` Step 3:

**Development**
| If the feature involves… | Add skill |
|---|---|
| Any code writing / refactoring | `code-writing-software-development` |
| Web app / landing page / UI | `build-website-web-app` |
| REST API design | `api-design` |
| Bug hunting / debugging | `systematic-debugging` |
| Branch merge / PR / cleanup | `branch-completion` |

**Planning & Architecture**
| If the feature involves… | Add skill |
|---|---|
| Requirements / architecture | `planning-specification-architecture` |
| Autonomous multi-step tasks | `autonomous-agents-task-automation` |
| Parallel agent orchestration | `parallel-agent-dispatch` |

**Testing & Quality**
| If the feature involves… | Add skill |
|---|---|
| Test-driven development | `tdd-workflow` |
| Security audit / OWASP review | `security-review` |
| Playwright / E2E testing | `e2e-testing` |

**Data & Backend**
| If the feature involves… | Add skill |
|---|---|
| PostgreSQL / Supabase | `postgres-patterns` |
| Database schema migrations | `database-migrations` |
| ClickHouse / OLAP analytics | `clickhouse-io` |

**Languages**
| If the feature introduces… | Add skill |
|---|---|
| Go / Golang | `golang-patterns` |
| Python / Django | `python-patterns` |
| Kotlin / Ktor | `kotlin-patterns` |
| Java / Spring Boot | `java-patterns` |
| Swift / SwiftUI / iOS | `swift-patterns` |
| C++ | `cpp-patterns` |
| Android / KMP | `android-patterns` |

**Claude / AI Platform**
| If the feature involves… | Add skill |
|---|---|
| Claude API / Anthropic SDK | `claude-developer-platform` |
| NotebookLM / podcast generation | `notebooklm` |

**UI & Design**
| If the feature involves… | Add skill |
|---|---|
| UI components / design system | `presentations-ui-design` |
| UI/UX design decisions | `ui-ux-pro-max` |
| shadcn/ui / Tailwind styling | `ui-styling` |
| Premium websites / cinematic animations | `cinematic-website-builder` |

**DevOps & Infrastructure**
| If the feature involves… | Add skill |
|---|---|
| Shell scripts / CI/CD / DevOps | `terminal-cli-devops` |
| Git worktree-based isolation | `git-worktrees` |

**Integrations**
| If the feature involves… | Add skill |
|---|---|
| Twitter/X API | `x-api` |
| WhatsApp messaging | `whatsapp-automation` |
| AI image/video generation | `fal-ai-media` |

### 4c — Agents (install silently if missing)

Check `.claude/agents/` junction exists. If absent, create it:
```bash
cmd /c mklink /J "PROJECT_ROOT\.claude\agents" "KIT_PATH\agents"
```

`refactor-cleaner` is always required. Check conditionally:
- E2E testing needed? → mention `e2e-runner` (available via junction)
- New language? → mention its reviewer (e.g., `python-reviewer`, `go-reviewer`)
- Database changes? → mention `database-reviewer`

Agents are available on-demand via the junction — no files to copy or register.

### 4d — Commands (install silently if missing)

Check `.claude/commands/` junction exists. If absent, create it:
```bash
cmd /c mklink /J "PROJECT_ROOT\.claude\commands" "KIT_PATH\commands"
```

Commands are available on-demand via the junction — no files to copy or register.

### 4e — Rules (install silently if missing)

`rules/common` is always required. Check junction exists. If absent, create it:
```bash
cmd /c mklink /J "PROJECT_ROOT\.claude\rules\common" "KIT_PATH\rules\common"
```

Add language-specific rule junctions only if a new language skill was added in Step 4b:

| New skill added | Additional junction |
|---|---|
| `golang-patterns` | `cmd /c mklink /J "PROJECT_ROOT\.claude\rules\golang" "KIT_PATH\rules\golang"` |
| `python-patterns` | `cmd /c mklink /J "PROJECT_ROOT\.claude\rules\python" "KIT_PATH\rules\python"` |
| `kotlin-patterns` | `cmd /c mklink /J "PROJECT_ROOT\.claude\rules\kotlin" "KIT_PATH\rules\kotlin"` |
| `build-website-web-app` or `java-patterns` | `cmd /c mklink /J "PROJECT_ROOT\.claude\rules\typescript" "KIT_PATH\rules\typescript"` |
| `swift-patterns` | `cmd /c mklink /J "PROJECT_ROOT\.claude\rules\swift" "KIT_PATH\rules\swift"` |

**If a junction target does not exist in the kit, skip silently.**

### 4f — Hooks & Contexts (install silently if missing)

Check `.claude/hooks/` and `.claude/contexts/` junctions exist. If absent, create them:
```bash
cmd /c mklink /J "PROJECT_ROOT\.claude\hooks"    "KIT_PATH\hooks"
cmd /c mklink /J "PROJECT_ROOT\.claude\contexts" "KIT_PATH\contexts"
```

### 4g — Present and Confirm

Build the list of new `@` imports and junctions, then present:

> "These components from the skill library are needed for your new feature. Should I add them?
> **Skills (new @imports):** [list]
> **Junctions (new):** [list or 'none']"

Only apply changes after the user confirms.

---

## 4b — Route to Department Lead Agent

After reading the existing codebase (Step 3) and the feature spec (Step 1), check if `.claude/CLAUDE.md` contains an `## Active Agent Team` section.

**If it exists:** identify which department lead handles this task type:
- Software / code / API / bug → `@software-cto`
- YouTube / blog / video / podcast → `@chief-content-officer`
- SEO / ads / email / growth → `@chief-marketing-officer`
- UI/UX / design / branding → `@chief-design-officer`
- AI / ML / LLM / agents → `@ai-cto`
- Product / SaaS / startup → `@chief-product-officer`
- Security / pentest / compliance → `@chief-security-officer`
- Database / infra / cloud / DevOps → `@software-cto`

Invoke the matching department lead agent with the context brief from Step 3 + the feature spec from Step 1. The lead agent will activate the right sub-agents and execute end-to-end without further prompting.

**If no `Active Agent Team` section exists:** continue with the standard planning workflow below (Step 5 onwards).

---

## 5. Create Feature Branch

Ask:
> "Should I create a git branch `feature/FEATURE_NAME` for this work?"

If yes, run `git checkout -b feature/FEATURE_NAME` from `PROJECT_ROOT`.

If the project has no git repo (`git rev-parse --is-inside-work-tree` fails), skip silently.

---

## 6. Create Feature Spec Folder

Create `.spec/FEATURE_NAME/` in `PROJECT_ROOT`.

This is the output directory for the planning skill. Each feature gets its own scoped subdirectory — do not write into root-level `.spec/` or overwrite specs from other features.

---

## 7. Update .claude/CLAUDE.md

Add or update an "Active Feature" section at the top of the "Start Here" block in `.claude/CLAUDE.md`:

```
## Active Feature
Feature: FEATURE_NAME
Spec: .spec/FEATURE_NAME/requirements.md, design.md
Tasks: .spec/FEATURE_NAME/tasks/
Current task: .spec/FEATURE_NAME/tasks/task-001.md
Branch: feature/FEATURE_NAME
```

This ensures the AI loads the right context immediately on the next session. The `Current task:` line is updated by `/task-handoff` after each task completes.

---

## 8. Update .github/copilot-instructions.md

If `PROJECT_ROOT/.github/copilot-instructions.md` exists, update it to mirror `.claude/CLAUDE.md`:

- For **new skills added in Step 4**: append the absolute kit path to the `## Skills — Read These Files for Coding Standards` section (one line per skill, matching the exact path used in the `@` import).
- Update the `## Active Feature` block to match Step 7.

**Rule:** Skill paths in `copilot-instructions.md` must match exactly the `@` imports in `.claude/CLAUDE.md`. Both files always list the same skill set.

GitHub Copilot Chat auto-loads this file for every request in the repository.

---

## 9. Synthesize Context Brief

Before invoking the planning skill, compile a brief covering:

- What the existing code already does (2–4 sentences)
- What the new spec adds or changes
- Which files and modules will be affected
- Constraints from the existing architecture (patterns, data models, API contracts already in use)
- Newly added `@` imported skills now available
- Output target: `.spec/FEATURE_NAME/`

---

## 10. Invoke planning-specification-architecture

Call the Skill tool with `planning-specification-architecture`.

Pass it:
1. The context brief from Step 9
2. The raw spec content from Step 1
3. The feature name and output path (`.spec/FEATURE_NAME/`)
4. Paths to existing `.spec/` files as brownfield context (the skill's "Handling Special Situations: greenfield vs. brownfield" section applies here)

The skill runs its 3-phase gated workflow:
- **Phase 1 — Requirements:** draft → present → user approves → save `.spec/FEATURE_NAME/requirements.md`
- **Phase 2 — Design:** draft → present → user approves → save `.spec/FEATURE_NAME/design.md`
- **Phase 3 — Tasks:** draft → present → user approves → save `.spec/FEATURE_NAME/tasks.md`

For large features spanning multiple PRs, the planning skill's Blueprint pipeline (Section 3) handles multi-PR decomposition.

**Rule:** Never write `requirements.md`, `design.md`, or `tasks.md` directly. These are produced only through the planning skill's gated workflow after explicit user approval of each phase.

### 10b. Run Task Enrichment

Immediately after `tasks.md` is saved (Phase 3 approved), run Task Enrichment (Section 12b of `planning-specification-architecture`):

1. Create `.spec/FEATURE_NAME/tasks/` directory.
2. For each task in `tasks.md`, create `task-NNN.md` (zero-padded, e.g. `task-001.md`).
3. Populate `## Session Bootstrap` with only the skills that task requires (from its `_Skills:_` line) — not all project skills.
4. Use `Glob` and `Grep` against `PROJECT_ROOT` source files — plus the context brief from Step 9 — to populate each task's Codebase Context section. **Embed actual code snippets** (interfaces, types, base classes, key constants) directly into the Key Code Snippets block, labeled with source path and line range. This eliminates file reading at execution time.
5. Copy Implementation Steps and Acceptance Criteria from the corresponding entry in `tasks.md`.
6. Leave all Handoff sections with placeholders — `/task-handoff` fills these during execution.

**Rule:** Only reference files that actually exist in `PROJECT_ROOT`. Never fabricate file paths.

---

## 11. Validate Changes

After all files are written and junctions created, verify each expected path exists.

**New @imports in `.claude/CLAUDE.md`** — each `@KIT_PATH/skills/.../SKILL.md` path must point to a file that actually exists in the kit. List any broken paths as warnings.

**Junctions** — verify that each junction created in Step 4 resolves correctly (e.g. `.claude/hooks/hooks.json` is readable through the junction). List any missing or broken junctions as warnings.

**Spec files** — verify the following exist:
- `.spec/FEATURE_NAME/requirements.md`
- `.spec/FEATURE_NAME/design.md`
- `.spec/FEATURE_NAME/tasks.md`
- `.spec/FEATURE_NAME/tasks/task-001.md`

Do not declare success until all checks pass.

---

## 12. Output Summary

After the gated workflow completes:

1. List all new files created under `.spec/FEATURE_NAME/`
2. State how many task files were created under `.spec/FEATURE_NAME/tasks/`
3. List newly added `@` imports and junctions (or "none needed")
4. State the active branch name
5. Tell the user: "Start with `.spec/FEATURE_NAME/tasks/task-001.md` — it is self-contained. Run `/task-handoff` after each task completes to propagate context to the next."
6. Remind the user: editing any skill in the kit takes effect on the next Claude Code session start. Editing agents/commands takes effect immediately.
7. If new MCP configs are needed, remind the user to replace `YOUR_TOKEN_HERE` with real API keys
