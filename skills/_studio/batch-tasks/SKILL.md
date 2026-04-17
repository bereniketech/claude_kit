---
name: batch-tasks
description: Run all tasks in a folder sequentially — for each task, load the ## Skills, ## Agents, and ## Commands declared in the task file, invoke the named agent with that context, run /verify, run /wrapup, then /clear before starting the next task.
---

# Batch Tasks

Execute every task in a tasks folder in sequence, with full quality gates and context reset between each task. Each task's declared skills, agents, and commands are loaded and passed into the agent invocation — the agent executes using exactly the tools the task specifies.

---

## 1. Locate the Tasks

Accept a folder path or file path as the argument. Default: `.spec/tasks/`.

| Input | How to read tasks |
|---|---|
| Folder of `.md` files | Sort files by name (`task-001.md`, `task-002.md`, ...) — each file is one task |
| Single `tasks.md` file | Each `- [ ] N.` checkbox block is one task |
| No argument | Use `.spec/tasks/` if it exists, else look for `.spec/tasks.md` |

**Rule:** If neither location exists, stop and tell the user — do not silently proceed.

---

## 2. Parse Each Task

For each task file, extract all of the following:

| Field | Where to find it | Default |
|---|---|---|
| Title | `# Task NNN: <title>` heading | `Task N` |
| Agent | `## Agents` section — first `@<agent-slug>` entry | `software-developer-expert` |
| Skills | `## Skills` section — all `- .kit/skills/...` lines | none |
| Commands | `## Commands` section — all `- /<command>` lines | none |
| Acceptance Criteria | `## Acceptance Criteria` section | — |
| Steps | `## Steps` section | — |
| Status | `Status: COMPLETE` in frontmatter or appended at end | incomplete |

**Rule:** Always read `## Skills`, `## Agents`, and `## Commands` from the task file itself. Never assume skills or agents from the previous task carry over — each task is self-contained.

---

## 3. Execution Loop

Process tasks in sorted order. For each task:

### 3a. Skip if complete

If the task has `Status: COMPLETE` in its frontmatter or appended at end, print:
```
[SKIP] Task N — Title (already complete)
```
Advance to the next task.

### 3b. Print task header

```
════════════════════════════════════════
TASK N / TOTAL: Title
Agent : <agent-slug>
Skills: <count> loaded
Commands: <list>
════════════════════════════════════════
```

### 3c. Load skills into context

Before invoking the agent, read every skill file listed in `## Skills`:

```
For each path in task's ## Skills:
  Read the SKILL.md file at that path
  Hold its content in context — the agent will use it
```

This is the same as loading skills via `@import` — the agent operates under the constraints and patterns defined in those skill files.

### 3d. Invoke the agent

Dispatch the primary agent from `## Agents` as a subagent. Pass the full task content plus all loaded skill content:

```markdown
## Loaded Skills
<concatenated content of all skill files from ## Skills>

## Task
<full task file content>

## Instructions
You are executing this task as @<agent-slug>.

Load and follow ALL skills listed above — they govern how you write code,
structure output, and enforce quality standards for this task.

Use the commands listed in ## Commands at the appropriate steps:
- /verify — run after implementation to check correctness
- /task-handoff — run when done to record decisions
- Other commands — run as directed in the task steps

Complete every step. Meet every acceptance criterion.
Do not ask questions — make decisions and document them in the handoff.
When done, output: TASK COMPLETE — <one-line summary of what was built>
```

Wait for the agent to finish before continuing.

### 3e. Run /verify

- **PASS** → continue to 3f
- **FAIL** → re-invoke the same agent with the verification errors appended to the task context (one retry)
  - **Still failing** → mark task BLOCKED, record the errors at the bottom of the task file, continue to next task:
    ```
    Status: BLOCKED — verify failed after retry
    Errors: <paste verification output>
    ```

### 3f. Mark task complete

Append to the bottom of the task file:
```
Status: COMPLETE
Completed: <ISO timestamp>
```

### 3g. Run /wrapup

Execute `/wrapup` now. This is mandatory after every task — do not skip it.

`/wrapup` saves session memories, records what was built, decisions made, and open questions, then pushes the session log to the NotebookLM brain if configured.

Do not proceed to 3h until `/wrapup` completes.

### 3h. Run /clear

Execute `/clear` to reset the context window before the next task. Skills, agent context, and implementation details from this task must not bleed into the next — each task starts fresh from its own `## Skills` and `## Agents` sections.

---

## 4. Final Summary

After all tasks are processed:

```
════════════════════════════════════════
BATCH COMPLETE
════════════════════════════════════════
Total   : N
Done    : X
Skipped : Y  (already complete)
Blocked : Z  (verify failed after retry)

Blocked tasks:
- Task 3 — Title (see task file for errors)

Next: open the first blocked task file to diagnose.
```

---

## 5. Rules

**Rule:** Tasks always run sequentially — never in parallel. Each task's output is the codebase the next task builds on.

**Rule:** Never skip /verify. A task is not complete until verification passes (or is explicitly marked BLOCKED after retry).

**Rule:** Load `## Skills` from the task file before invoking the agent — the agent must operate under those skill constraints, not generic defaults.

**Rule:** The primary agent to invoke is the first entry in the task's `## Agents` section. If the task lists multiple agents (e.g., `@ai-ml-expert` + `@technical-writer-expert`), the first is the executor; the rest are reviewers — invoke them after the primary agent completes, before /verify.

**Rule:** `--from N` starts the loop at task N, skipping earlier tasks regardless of status. `--only N` runs exactly one task.

**Rule:** If `## Skills`, `## Agents`, or `## Commands` sections are missing from a task file, stop and warn — do not proceed with that task until the sections are present.
