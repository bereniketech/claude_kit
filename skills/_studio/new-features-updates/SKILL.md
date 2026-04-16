---
name: new-features-updates
description: Plan and scaffold new features onto an existing project. Analyzes the current codebase, ingests specs (text or folder of docs/images), installs missing skill library components via @imports and junctions (zero-copy), creates a feature branch and spec folder, then delegates to planning-specification-architecture-software (or the appropriate company planning skill) for the gated requirements → design → tasks workflow.
---

## HARD STOP — READ THIS BEFORE ANYTHING ELSE

**THIS SKILL HAS EXACTLY TWO JOBS:**
1. Analyze the existing codebase (read-only, no writes, no todos)
2. Invoke the `@company-coo` board agent with a brief

**EVERYTHING ELSE IS A VIOLATION.** The following are explicitly forbidden at every step:
- Writing any file (CSS, JS, HTML, MD, JSON — anything)
- Creating todos
- Creating a plan, spec, or design
- Selecting specialists or agents
- Writing code of any kind
- Starting execution of any task
- Skipping the board agent invocation

**If you feel the urge to write code or skip to implementation — STOP. Invoke the board agent instead.**

A previous assistant violated this skill by reading the codebase and immediately writing CSS. That must never happen again. The board gate is not optional. The planning gate is not optional. User approval at each gate is not optional.

---

## CRITICAL: Hierarchy Enforcement

**THIS SKILL IS ANALYSIS + ROUTING ONLY.** It must NEVER create plans, select specialists, or execute code. It routes to the board, and the board routes through CEOs to specialists. Plans originate from specialists, never from board members.

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

**HARD STOP:** After gathering specs, do NOT begin planning. Proceed to Step 2.

---

## 2. Confirm Project Root

Check if CWD contains existing project files (`package.json`, `pyproject.toml`, `go.mod`, source folders).

If CWD is the skill library (has `.claude/skills/*/SKILL.md`), ask:
> "What is the path to your existing project?"

Set `PROJECT_ROOT`. Read `PROJECT_ROOT/.claude/project-config.md` to get `KIT_PATH` (the absolute path to the skill library) — needed in Step 5.

**Rule:** Never write any file outside `PROJECT_ROOT`. Never treat the skill library directory as the project.

**HARD STOP:** After confirming project root, do NOT write files or create specs. Proceed to Step 3.

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

**HARD STOP:** After codebase analysis, do NOT begin writing code or selecting specialists. Proceed to Step 3b or Step 4.

### 3b — Conditional Architect Analysis

Only if the feature spec explicitly involves one or more of:
- A new service or module boundary
- A change to the existing data model
- Integration with a new external system

…then call the `@architect` agent with: the existing codebase summary from Step 3 + the feature spec from Step 1. Use its output to inform the spec structure in Step 6.

**Otherwise: skip. Do not call @architect for features that extend existing patterns.**

---

## 4. Do NOT Set Up Infrastructure (Board Decides)

**SKIP infrastructure setup entirely.** Do NOT create junctions, do NOT initialize git, do NOT write any files.

Simply read `KIT_PATH` from `PROJECT_ROOT/.claude/project-config.md` (if it exists). Pass this to the board in Step 5.

The board will handle all infrastructure setup: git initialization, junction creation, CLAUDE.md updates, etc.

**HARD STOP:** Do not write anything. Do not create todos. Proceed directly to Step 5.

---

## 5. Invoke the Board Agent — MANDATORY, NON-NEGOTIABLE

**THIS IS NOT OPTIONAL. THIS MUST HAPPEN BEFORE ANY CODE IS WRITTEN.**

The skill has exactly one output: the `Agent` tool call invoking the board. If you complete Steps 1–4 and do not call the board agent, **the skill has failed** — regardless of how much analysis was done.

**Do not:**
- Write code "while the board agent runs"
- Create todos "to help track the work"
- Summarize what you plan to implement
- Begin any file writes
- Select which specialists will work on this

All of that happens downstream, after the board → CEO → specialist → planning gates → user approval chain runs.

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
  description: "Route [feature name] to board for assessment and CEO delegation",
  prompt: "[Board member], here is a new feature request:
  
  **Project:** [project name at PROJECT_ROOT]
  **Feature:** [FEATURE_NAME]
  **Spec:** [raw spec text from Step 1]
  
  **Codebase Summary:**
  [folder structure, tech stack, key files]
  
  **Existing Tasks Found:**
  [list .spec/ file paths and their status, or "none"]
  
  **Infrastructure:** Junctions ready, CLAUDE.md exists at KIT_PATH = [KIT_PATH]
  
  Your role: 
  1. Assess the feature scope
  2. Determine which operating company CEO should own this work
  3. Route this feature request to that CEO (not to specialists directly)
  4. The CEO will delegate to appropriate specialists
  5. Each specialist MUST invoke skills/planning/planning-specification-architecture-software/SKILL.md first — producing requirements.md → design.md → tasks/*.md with explicit user approval at each gate — before any execution begins
  
  DO NOT create a plan yourself. DO NOT call specialist agents directly. Always route through the company CEO. Planning always goes through the appropriate company planning skill (planning-specification-architecture-software / -marketing / -media) — never custom or abbreviated flows."
)
```

**The skill ends when the Agent tool is called.** Nothing after this point is the skill's responsibility.

**HARD STOP:** If you have reached this point without calling the Agent tool, call it now. Do not proceed to any other action.

---

## What Correct Execution Looks Like

```
Step 1: Read spec from user  →  NO code written
Step 2: Confirm project root  →  NO code written  
Step 3: Analyze codebase  →  NO code written
Step 4: Read KIT_PATH  →  NO code written
Step 5: Call Agent(company-coo, ...)  →  SKILL COMPLETE
```

After Step 5, the chain runs autonomously:
```
company-coo → identifies company → routes to CEO
CEO → assesses request → delegates to specialists
specialists → run planning skill → requirements.md [USER APPROVES]
                                 → design.md [USER APPROVES]
                                 → tasks/*.md [USER APPROVES]
                                 → execution begins
```

**No user approval = no execution. This is enforced by the planning skill, not by this skill.**

---

## What a Violation Looks Like (NEVER DO THIS)

```
❌ Read the codebase → start writing CSS  
❌ Understand the feature → create a todo list for implementation  
❌ Analyze files → summarize what you "will now implement"  
❌ Skip the Agent call → go straight to file writes  
❌ Call the Agent AND also start writing code in parallel  
❌ Decide "the user wants it fast so I'll skip the gate"  
```

Any of these is a failure of this skill. The user's instruction to "do it" does not override the planning gate. Speed is never a justification for skipping approval.

---

## Rules

### Hierarchy (CRITICAL)
- **ALWAYS route to the board.** The board (company-coo in most cases) decides which company CEO should lead, then CEOs delegate to specialists.
- **NEVER create a plan yourself.** Plans originate from specialists delegated by company CEOs, never from routing-level agents.
- **NEVER invoke specialist agents directly.** Always route through the board and company CEOs.
- **Analysis + routing only.** Read the codebase, gather specs, verify infrastructure (junctions, CLAUDE.md stub). Then invoke the board. Stop.

### Scope Boundaries
- **In scope:** Analyze feature spec + existing codebase, confirm project root, verify junctions, read existing tasks
- **Out of scope:** Planning, specialist selection, code changes, task execution, skill selection, creating .spec/ files, creating todos, writing any file of any type
