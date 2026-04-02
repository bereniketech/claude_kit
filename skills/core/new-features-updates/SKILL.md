---
name: new-features-updates
description: Plan and scaffold new features onto an existing project. Analyzes the current codebase, ingests specs (text or folder of docs/images), installs missing skill library components, creates a feature branch and spec folder, then delegates to planning-specification-architecture for the gated requirements → design → tasks workflow.
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

Set `PROJECT_ROOT`. Read `PROJECT_ROOT/.claude/project-config.md` to get the skill library path — needed in Step 4.

**Rule:** Never write any file outside `PROJECT_ROOT`. Never treat the skill library directory as the project.

---

## 3. Analyze Existing Codebase

Use `Glob` and `Grep` to understand what already exists in `PROJECT_ROOT`:

- Top-level folder structure
- Tech stack (infer from `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, etc.)
- Key source files: entry points, API routes, data models, main modules
- Existing `.spec/` folders and files — read all present to understand prior plans
- Already-installed components: list `.claude/skills/`, `.claude/agents/`, `.claude/commands/`
- Read `.claude/CLAUDE.md` to see what is currently registered

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

### Baseline mandatory skills (always install if absent)

Regardless of what the new feature involves, every project must have all eight of these. Check first and install silently if missing — no user confirmation needed:

| Skill | Outcome it guarantees |
|---|---|
| `code-writing-software-development` | Consistency — coding standards, read-before-edit discipline, 6-phase verification loop on every change |
| `tdd-workflow` | Higher test coverage — 80%+ enforced, Red/Green/Refactor, CI gate before every PR |
| `security-review` | Fewer code review issues — OWASP Top 10 checklist before any feature touches auth, input, secrets, or APIs |
| `autonomous-agents-task-automation` | Faster feature completion + less context switching — parallel execution, model routing, subagent delegation |
| `continuous-learning` | Knowledge persistence — patterns and solutions survive across sessions, no repeated debugging |
| `strategic-compact` | Context rot prevention — compacts at phase boundaries so architectural decisions stay coherent |
| `notebooklm` | Second brain — session knowledge is queryable; "check Brain first" reduces redundant codebase scanning and token usage |
| `wrapup` | Session persistence — captures decisions, learnings, and open threads at session end; pushes to NotebookLM Brain |

Copy each missing skill from the skill library and register it in `.claude/CLAUDE.md` under the Skills section.

### Conditional skills

Using the skill library path from `project-config.md`, compare the new feature requirements against already-installed components. Apply the same selection logic as `generate-claude-md` Step 3.

**Skills** — cross-check `.claude/skills/` against what the new feature needs:
- New language introduced? → add language skill (`golang-patterns`, `python-patterns`, `kotlin-patterns`, etc.)
- New testing requirements? → add `tdd-workflow`, `e2e-testing`, or `security-scan`
- New data/backend work? → add `postgres-patterns`, `database-migrations`, or `clickhouse-io`
- New integrations? → add the matching integration skill
- AI/LLM work? → add `claude-developer-platform`

**Agents** — also check these mandatory agents and install silently if missing:
`refactor-cleaner` (always required) — then check conditionally:
- E2E testing needed? → add `e2e-runner`
- New language? → add its reviewer (e.g., `python-reviewer`, `go-reviewer`)
- Database changes? → add `database-reviewer`

**Commands** — also check these mandatory commands and install silently if missing:
`plan`, `tdd`, `verify`, `quality-gate`, `refactor-clean`, `save-session`, `resume-session` (all always required) — then check conditionally:
- E2E workflows? → add `e2e.md`
- Language-specific builds? → add relevant build/test commands

**Rules** — if `rules/common/context-budget.md` is absent, copy it silently from the skill library.

**Hooks** — if `.claude/hooks/hooks.json` is absent, offer to copy from the skill library.

Build the list of missing components, then present it:
> "These components from the skill library are needed for your new features. Should I install them? [list]"

Only copy after the user confirms. For each installed skill/agent/command, update `.claude/CLAUDE.md` to register it under the appropriate section.

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

## 8. Synthesize Context Brief

Before invoking the planning skill, compile a brief covering:

- What the existing code already does (2–4 sentences)
- What the new spec adds or changes
- Which files and modules will be affected
- Constraints from the existing architecture (patterns, data models, API contracts already in use)
- Newly installed skills and agents now available
- Output target: `.spec/FEATURE_NAME/`

---

## 9. Invoke planning-specification-architecture

Call the Skill tool with `planning-specification-architecture`.

Pass it:
1. The context brief from Step 8
2. The raw spec content from Step 1
3. The feature name and output path (`.spec/FEATURE_NAME/`)
4. Paths to existing `.spec/` files as brownfield context (the skill's "Handling Special Situations: greenfield vs. brownfield" section applies here)

The skill runs its 3-phase gated workflow:
- **Phase 1 — Requirements:** draft → present → user approves → save `.spec/FEATURE_NAME/requirements.md`
- **Phase 2 — Design:** draft → present → user approves → save `.spec/FEATURE_NAME/design.md`
- **Phase 3 — Tasks:** draft → present → user approves → save `.spec/FEATURE_NAME/tasks.md`

For large features spanning multiple PRs, the planning skill's Blueprint pipeline (Section 3) handles multi-PR decomposition.

**Rule:** Never write `requirements.md`, `design.md`, or `tasks.md` directly. These are produced only through the planning skill's gated workflow after explicit user approval of each phase.

### 9b. Run Task Enrichment

Immediately after `tasks.md` is saved (Phase 3 approved), run Task Enrichment (Section 12b of `planning-specification-architecture`):

1. Create `.spec/FEATURE_NAME/tasks/` directory.
2. For each task in `tasks.md`, create `task-NNN.md` (zero-padded, e.g. `task-001.md`).
3. Populate `## Session Bootstrap` with only the skills that task requires (from its `_Skills:_` line) — not all project skills.
4. Use `Glob` and `Grep` against `PROJECT_ROOT` source files — plus the context brief from Step 8 — to populate each task's Codebase Context section. **Embed actual code snippets** (interfaces, types, base classes, key constants) directly into the Key Code Snippets block, labeled with source path and line range. This eliminates file reading at execution time.
5. Copy Implementation Steps and Acceptance Criteria from the corresponding entry in `tasks.md`.
6. Leave all Handoff sections with placeholders — `/task-handoff` fills these during execution.

**Rule:** Only reference files that actually exist in `PROJECT_ROOT`. Never fabricate file paths.

---

## 10. Output Summary

After the gated workflow completes:

1. List all new files created under `.spec/FEATURE_NAME/`
2. State how many task files were created under `.spec/FEATURE_NAME/tasks/`
3. List newly installed skills, agents, and commands (or "none needed")
4. State the active branch name
5. Tell the user: "Start with `.spec/FEATURE_NAME/tasks/task-001.md` — it is self-contained. Run `/task-handoff` after each task completes to propagate context to the next."
6. If new MCP configs were added, remind the user to replace `YOUR_TOKEN_HERE` with real API keys
