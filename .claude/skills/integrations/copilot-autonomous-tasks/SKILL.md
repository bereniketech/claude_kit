---
name: copilot-autonomous-tasks
description: Run GitHub Copilot in VSCode as a zero-interruption autonomous agent — load skills, implement the current task file, verify, hand off, and advance to the next task without human intervention.
---

# Copilot Autonomous Tasks

Use this skill when GitHub Copilot agent mode should complete a project's task queue end-to-end: read task → implement → verify → hand off → clear → next task, with no mid-task questions or manual approvals.

---

## 1. VSCode Setup (One-Time)

Create `.vscode/settings.json` at the project root (commit this file):

```json
{
  "github.copilot.chat.agent.autoApprove": true,
  "github.copilot.chat.agent.terminal.allowList": ["*"]
}
```

Switch the Copilot Chat input to **Agent** mode via the dropdown next to the chat input.

**Rule:** `"terminal.allowList": ["*"]` allows every terminal command without prompting. Tighten to `["npm *", "git *", "npx *"]` if the project warrants tighter control.

---

## 2. copilot-instructions.md — Autonomous Loop Directive

The loop lives in `.github/copilot-instructions.md`, not the activation prompt. Copilot auto-loads this file on every new chat, so the loop persists across `/clear` context resets.

Replace `## Start Here` in every project's `copilot-instructions.md` with:

```markdown
## Autonomous Loop — Begin Immediately on Session Start
On session start, do not wait for user input. Start from step 1 now.

1. Read `Current task:` from `## Active Feature` above. If it reads `ALL TASKS COMPLETE`, stop and notify the user.
2. Open that task file — it is self-contained with all acceptance criteria.
3. Read the skill files listed in `## Skills` above (once per session, not per task).
4. Implement all acceptance criteria. Zero clarifying questions mid-task — resolve unknowns from the task file, skills, or existing code. If genuinely blocked, write `BLOCKED: <reason>` in the task file's `## Status` and stop.
5. Run `/verify` — all phases must pass before continuing. On failure, fix the root cause. If the same phase fails 3× in a row, write `VERIFY_FAIL: <phase> — <error>` in `## Status` and stop.
6. Run `/task-handoff` — updates `Current task:` in this file and commits.
7. Run `/clear` — mandatory between every task to reset context. This file auto-loads in the new chat; the loop restarts from step 1 automatically.
```

**Rule:** The "Begin Immediately on Session Start" header is what drives auto-continuation after `/clear`. Copilot re-reads this file in the new chat and starts the loop without any user re-prompt.

**Rule:** Always read `Current task:` from `copilot-instructions.md` — it is auto-loaded and always reflects the latest pointer after `/task-handoff`.

---

## 3. Activation Prompt

Paste this once to start a session. The loop then runs autonomously:

```
Read .github/copilot-instructions.md. Follow the ## Autonomous Loop instructions exactly.
```

That's all. The full loop logic is in `copilot-instructions.md` and persists across context resets.

---

## 4. Behavioral Rules

When operating as an autonomous Copilot agent:

- **Never ask a clarifying question mid-task.** Resolve unknowns from the task file, skill files, or existing code. If genuinely unresolvable, write `BLOCKED: <reason>` in the task file's `## Status` section and stop.
- **Never skip `/verify`.** All 6 phases must pass before `/task-handoff`.
- **On verify failure:** fix the root cause. If the same phase fails 3 consecutive times, write `VERIFY_FAIL: <phase> — <error summary>` in `## Status` and stop. Do not commit.
- **On destructive operations** (drop table, force push to main, delete untracked files): stop and ask the user explicitly. Never auto-approve destructive ops regardless of agent mode settings.
- **Always `/clear` between tasks.** Mandatory after every `/task-handoff`. `copilot-instructions.md` auto-loads in the new chat and restarts the loop.

**Rule:** Never modify test files to make tests pass. Fix the code under test.

---

## 5. Loading Skills

`.github/copilot-instructions.md` lists absolute skill file paths under `## Skills`. Read each listed skill file before writing any code. Skills contain the coding standards, test patterns, and security rules for this codebase.

Do NOT use `@` imports — Copilot cannot resolve them. Use the explicit paths in `## Skills` only.

---

## 6. Termination Conditions

Stop the loop and notify the user when:

| Condition | Action |
|---|---|
| `Current task: ALL TASKS COMPLETE` | Output "ALL TASKS COMPLETE." Stop. |
| Same verify phase fails 3× in a row | Write `VERIFY_FAIL` to task file. Stop. Output failure summary. |
| Task file `## Status` is `BLOCKED` | Output "BLOCKED on \<task-file\>: \<reason\>". Stop. |
| Required env var is missing | Output "MISSING: \<VAR\> needed for \<task\>". Stop. |
| Destructive operation required | Stop. Ask the user before proceeding. |

---

## 7. After Each Task — Clear and Continue

After `/task-handoff` commits:

1. Run `/clear` — mandatory. Resets stale context from the completed task.
2. Copilot auto-loads `copilot-instructions.md` in the new chat.
3. The "Begin Immediately on Session Start" directive triggers the loop from step 1 — no user re-prompt needed.

**Rule:** Do not re-paste the activation prompt after `/clear`. `copilot-instructions.md` carries the full loop and auto-starts it.

---

## 8. Resume Mid-Task

If interrupted (context window hit, network error, manual stop), paste this to resume:

```
Read .claude/CLAUDE.md → Current task. Read that task file. Check ## Status for any
BLOCKED notes or partial work. Continue from where work stopped. When done, run
/verify then /task-handoff.
```

If the task file has no status notes, run `git status` and check recent file modification times to reconstruct where implementation stopped before resuming.
