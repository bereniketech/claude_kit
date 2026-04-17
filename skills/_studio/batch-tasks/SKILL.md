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

### 3g. Inline wrapup (mandatory — execute every step)

Do NOT call `/wrapup` as a command — it will not execute. Run these steps directly:

**3g-1. Verify NotebookLM CLI:**
```bash
export PATH="$HOME/bin:$PATH"
notebooklm --help > /dev/null 2>&1 && notebooklm auth check
```
If CLI missing or auth fails, skip NotebookLM steps (3g-2 through 3g-5) but continue with memory saves.

**3g-2. Ensure AI Brain notebook exists:**
```bash
REPO=$(basename "$(git rev-parse --show-toplevel 2>/dev/null || basename "$PWD")")
notebooklm list --json
```
Find notebook named `<repo> AI Brain`. If not found, create it:
```bash
notebooklm create "${REPO} AI Brain" --json
```
Save the notebook ID to `memory/reference_brain_notebook.md` and update `MEMORY.md`.

**3g-3. Write task session summary** to `/tmp/task-summary-NNN.md`:
```markdown
# Task NNN Summary — <ISO date>

## Task
<title from task file>

## What Was Built
<bullet points of files created/modified>

## Decisions Made
<key decisions and reasoning>

## Open Threads
<anything unresolved>
```

**3g-4. Push summary to AI Brain:**
```bash
notebooklm use <BRAIN_NOTEBOOK_ID>
notebooklm source add /tmp/task-summary-NNN.md --notebook <BRAIN_NOTEBOOK_ID>
```

**3g-5. Save memories** — check existing memory index, then save/update:
- `project` memory if work or decisions affect future tasks
- `feedback` memory if the agent made a non-obvious choice
- Do not duplicate existing memories — update them

Do not advance to 3h until 3g-1 through 3g-5 are complete.

### 3h. Clear context (mandatory — execute every step)

Do NOT call `/clear` as a command. Run these steps directly:

**3h-1.** Record the next task number to resume at.
**3h-2.** Drop all in-context information from this task (code, decisions, skill content).
**3h-3.** Re-read this batch-tasks SKILL.md from disk.
**3h-4.** Re-read the next task file from disk — do not use anything cached from the previous task.
**3h-5.** Continue the execution loop from step 3a for the next task.

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

**Rule:** Never skip wrapup (step 3g). All 5 sub-steps must run — CLI check, brain notebook, summary file, NotebookLM push, memory saves. `/wrapup` as a slash command does not self-execute — the steps must be run inline.

**Rule:** Never skip context clear (step 3h). After wrapup, drop all task context, re-read this skill from disk, re-read the next task file from disk before starting it.

**Rule:** Load `## Skills` from the task file before invoking the agent — the agent must operate under those skill constraints, not generic defaults.

**Rule:** The primary agent to invoke is the first entry in the task's `## Agents` section. If the task lists multiple agents (e.g., `@ai-ml-expert` + `@technical-writer-expert`), the first is the executor; the rest are reviewers — invoke them after the primary agent completes, before /verify.

**Rule:** `--from N` starts the loop at task N, skipping earlier tasks regardless of status. `--only N` runs exactly one task.

**Rule:** If `## Skills`, `## Agents`, or `## Commands` sections are missing from a task file, stop and warn — do not proceed with that task until the sections are present.
