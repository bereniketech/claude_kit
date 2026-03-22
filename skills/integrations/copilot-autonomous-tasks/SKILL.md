---
name: copilot-autonomous-tasks
description: Run GitHub Copilot in VSCode as a zero-interruption autonomous agent — load skills, implement the current task file, verify, hand off, and advance to the next task without human intervention.
---

# Copilot Autonomous Tasks

Use this skill when GitHub Copilot agent mode should complete a project's task queue end-to-end: read task → implement → verify → hand off → clear → next task, with no mid-task questions or manual approvals.

---

## 1. VSCode Setup (One-Time)

Add to `.vscode/settings.json` (project-level, commit this file):

```json
{
  "github.copilot.chat.agent.autoApprove": true,
  "github.copilot.chat.agent.terminal.allowList": ["*"]
}
```

Switch the Copilot Chat input to **Agent** mode via the dropdown next to the chat input.

**Rule:** `"terminal.allowList": ["*"]` allows every terminal command without prompting. Tighten to `["npm *", "git *", "npx *"]` if the project warrants tighter control.

---

## 2. Activation Prompt

Paste this into the Copilot Chat input to start an autonomous session:

```
Read .github/copilot-instructions.md and .claude/CLAUDE.md. Find Current task under
## Active Feature in .claude/CLAUDE.md. Read that task file. Implement it autonomously
— zero questions mid-task. When done, run /verify then /task-handoff, then /clear,
then repeat for the next task.
```

Copilot will:
1. Read `.github/copilot-instructions.md` for the skill list and project context.
2. Read `.claude/CLAUDE.md` → `## Active Feature > Current task:` for the task file path.
3. Read the task file (e.g. `.spec/tasks/task-002.md`) for acceptance criteria.
4. Implement all acceptance criteria — no clarifying questions mid-task.
5. Run `/verify` — execute all 6 phases (build, types, lint, tests, secrets, git status).
6. Run `/task-handoff` — update handoff notes, advance the pointer, commit.
7. Run `/clear` — wipe context window and screen.
8. Read the new `Current task:` from `.claude/CLAUDE.md` and repeat from step 1.

**Rule:** Always read `.claude/CLAUDE.md` for `Current task:` — not `copilot-instructions.md`, which becomes stale after `/task-handoff` updates the pointer.

---

## 3. Behavioral Rules

When operating as an autonomous Copilot agent:

- **Never ask a clarifying question mid-task.** Resolve unknowns from the task file, skill files, or existing code. If genuinely unresolvable, write `BLOCKED: <reason>` in the task file's `## Status` section and stop.
- **Never skip `/verify`.** All 6 phases must pass before `/task-handoff`.
- **On verify failure:** fix the root cause. If the same phase fails 3 consecutive times, write `VERIFY_FAIL: <phase> — <error summary>` in `## Status` and stop. Do not commit.
- **On destructive operations** (drop table, force push to main, delete untracked files): stop and ask the user explicitly. Never auto-approve destructive ops regardless of agent mode settings.
- **Always `/clear` between tasks.** Accumulated context from a completed task degrades output quality on the next one.

**Rule:** Never modify test files to make tests pass. Fix the code under test.

---

## 4. Loading Skills

`.github/copilot-instructions.md` lists absolute skill file paths under `## Skills`. Read each listed skill file before writing any code. Skills contain the coding standards, test patterns, and security rules for this codebase.

Do NOT use `@` imports — Copilot cannot resolve them. Use the explicit paths in `## Skills` only.

---

## 5. Termination Conditions

Stop the loop and notify the user when:

| Condition | Action |
|---|---|
| `Current task: ALL TASKS COMPLETE` | Output "ALL TASKS COMPLETE." Stop. |
| Same verify phase fails 3× in a row | Write `VERIFY_FAIL` to task file. Stop. Output failure summary. |
| Task file `## Status` is `BLOCKED` | Output "BLOCKED on \<task-file\>: \<reason\>". Stop. |
| Required env var is missing | Output "MISSING: \<VAR\> needed for \<task\>". Stop. |
| Destructive operation required | Stop. Ask the user before proceeding. |

---

## 6. After Each Task — Clear and Continue

After each `/task-handoff` commit:

1. Run `/clear` — clears context window and screen.
2. Read `.claude/CLAUDE.md` → `## Active Feature > Current task:` for the next task path.
3. Begin the cycle again from Section 2.

In Claude Code agent mode, `/clear` is a built-in command. In GitHub Copilot Chat, click **New Chat** to achieve the same context reset, then paste the activation prompt from Section 2.

**Rule:** Always clear before the next task. Context from a completed task contains stale file state and resolved-but-remembered errors that pollute the next implementation.

---

## 7. Resume Mid-Task

If interrupted (context window hit, network error, manual stop), paste this to resume:

```
Read .claude/CLAUDE.md → Current task. Read that task file. Check ## Status for any
BLOCKED notes or partial work. Continue from where work stopped. When done, run
/verify then /task-handoff.
```

If the task file has no status notes, run `git status` and check recent file modification times to reconstruct where implementation stopped before resuming.
