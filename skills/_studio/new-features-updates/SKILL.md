---
name: new-features-updates
description: Plan and scaffold new features onto an existing project. Analyzes the current codebase, ingests specs (text or folder of docs/images), installs missing skill library components via @imports and junctions (zero-copy), creates a feature branch and spec folder, then delegates to planning-specification-architecture for the gated requirements → design → tasks workflow.
---

# New Features & Updates

Use this skill when new specifications arrive for an existing project. It bridges the gap between `generate-claude-md` (used once at project start) and active feature development.

**CRITICAL:** This skill is analysis + routing ONLY. It does not execute code, create tasks, or invoke execution agents. It gathers information and hands off to the board. If you find yourself about to invoke an agent to execute something, you have left the skill's scope.

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
- Existing `.spec/` folders and files — read all present to understand prior plans. If task files exist, note their status and include them in the board brief (Step 5). **Never execute or continue any task yourself — existing task files are context for the board, not a signal to self-execute.**
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

## 5. Invoke the Board Agent

**STOP HERE. This is the skill's end point.** Do not proceed beyond this section. Do not write code. Do not create todos. Do not edit files. Do not invoke execution agents. Do not execute tasks. Do not spawn subagents.

The ONLY action the skill performs in Step 5 is invoking a board agent (company-coo, chief-design-officer, etc.). Nothing else.

### 5a — Choose Board Entry Point

Route based on feature scope:

| Feature scope | Invoke |
|---|---|
| Code, software, website, AI, infra, product, or multi-company | `@company-coo` |
| Design coherence: UI + brand + visuals | `@chief-design-officer` |
| HR, hiring, comp, performance, contracts | `@people-operations-expert` |
| Comms, decision tracking, weekly reviews | `@chief-of-staff` |

Default: `@company-coo`

### 5b — Prepare Board Brief

Collect (do not expand on):
- **Codebase:** folder structure, tech stack, key files from Step 3
- **Feature:** the spec + `FEATURE_NAME` from Step 1
- **Existing tasks:** list any `.spec/*/tasks/task-*.md` files found; mark as "pending" or "done"
- **Infrastructure:** junctions verified, `.claude/CLAUDE.md` exists, `KIT_PATH` known
- **Output:** `.spec/FEATURE_NAME/` for artifacts, `.spec/FEATURE_NAME/tasks/` for tasks

### 5c — Invoke Agent NOW

Use the Agent tool:
```
Agent(
  description: "Route [feature name] to board for planning and execution",
  prompt: "[Board member], here is a new feature request:
  
  **Project:** [project name at PROJECT_ROOT]
  **Feature:** [FEATURE_NAME]
  **Spec:** [raw spec text from Step 1]
  
  **Codebase Summary:**
  [folder structure, tech stack, key files]
  
  **Existing Tasks Found:**
  [list .spec/ file paths and their status, or "none"]
  
  **Infrastructure:** Junctions ready, CLAUDE.md exists at KIT_PATH = [KIT_PATH]
  
  Your role: assess the feature, select required skills, plan (requirements → design → tasks → enrichment), execute task-001 → cascade via /task-handoff."
)
```

**The skill ends when the Agent tool is called.** Nothing after this point is the skill's responsibility.

**Rule:** If the board agent is not invoked by the end of this section, the skill has failed. Never proceed to code, todos, or task execution without invoking the board.
