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

## 4. Do NOT Add Skills Yet (Board Decides)

Read `KIT_PATH` from `PROJECT_ROOT/.claude/project-config.md`. Do NOT add or select skills here — the board will decide which skills are needed after assessing the feature spec.

Simply confirm that junctions are ready and that `.claude/CLAUDE.md` exists (board will update it with selected skills).

### 4c — Verify Junctions Exist

Junctions should already exist from project generation. Verify:
```bash
cmd /c mklink /J "PROJECT_ROOT\.claude\agents"   "KIT_PATH\agents"
cmd /c mklink /J "PROJECT_ROOT\.claude\commands" "KIT_PATH\commands"
cmd /c mklink /J "PROJECT_ROOT\.claude\hooks"    "KIT_PATH\hooks"
cmd /c mklink /J "PROJECT_ROOT\.claude\contexts" "KIT_PATH\contexts"
cmd /c mklink /J "PROJECT_ROOT\.claude\rules\common" "KIT_PATH\rules\common"
```

Create any that are missing. Agents, commands, rules, hooks, and contexts are now available on-demand via the junctions.

---

## 5. Route to the Board

**This is the last step the skill performs.** Feature branch, planning (requirements → design → tasks → task enrichment), and execution are all owned by the board. Do not do any of those things yourself.

### 5a — Identify the right board entry point

| If the feature is about… | Start with |
|---|---|
| Code, software, website, AI, infra, product, or spans multiple companies | `@company-coo` |
| Design coherence across software UI + marketing brand + media visuals | `@chief-design-officer` |
| Hiring, comp, performance reviews, contracts, handbook | `@people-operations-expert` |
| Comms triage (email, Slack, LINE, Messenger), decision tracking, weekly reviews | `@chief-of-staff` |

For anything ambiguous, default to `@company-coo` — the COO routes to board peers when needed.

### 5b — Invoke the board member with a complete brief

Invoke immediately after Step 4c infrastructure is confirmed. Pass them:

1. **Codebase summary** — what the existing code does (from Step 3), tech stack, key files and modules
2. **Feature spec** — the raw spec from Step 1 + `FEATURE_NAME`
3. **Infrastructure ready** — junctions confirmed in Step 4c; `.claude/CLAUDE.md` exists (board will select and add skills)
4. **Output target** — `.spec/FEATURE_NAME/` for all planning artifacts, `.spec/FEATURE_NAME/tasks/` for Haiku-ready task files
5. **Board's responsibilities from here:**
   - Decide which CEO (or board peer) should lead this feature
   - Select only the skills that company needs and add them to `.claude/CLAUDE.md`
   - Drive all planning (requirements → design → tasks → task enrichment)
   - Execute task-001 and cascade via `/task-handoff`

The board routes internally. Do not prescribe which CEO or specialist to use.

**Rule:** This step is mandatory and fires immediately after Step 4c. Never bypass the board to call a CEO or specialist directly.
