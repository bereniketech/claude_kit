---
name: nanoclaw-repl
description: Operate and extend NanoClaw v2, ECC's zero-dependency session-aware REPL built on claude -p.
---

# NanoClaw REPL

Use this skill when running or extending `scripts/claw.js` — ECC's zero-dependency, markdown-backed REPL for persistent Claude Code sessions.

> Note: The core patterns from this skill have been merged into `autonomous-agents-task-automation`. Refer to that skill for broader session and multi-agent orchestration guidance.

---

## 1. Core Capabilities

| Command | Purpose |
|---------|---------|
| `/model` | Switch active model |
| `/load` | Dynamically load a skill into the session |
| `/branch` | Branch the session before high-risk changes |
| `/compact` | Compact history after major milestones |
| `/search` | Cross-session search |
| `/export` | Export session to md / json / txt |
| `/metrics` | View session metrics |

---

## 2. Operating Guidance

Use sessions with a single focused task. Branch before any high-risk or irreversible operation. Compact after major milestones to keep context manageable. Export before sharing or archiving a session.

**Rule:** Never let a session accumulate multiple unrelated tasks — branch or start a new session instead.

---

## 3. Extension Rules

When adding new commands or modifying `claw.js`:

- Keep zero external runtime dependencies — no npm packages in the REPL core.
- Preserve markdown-as-database compatibility — session files must remain valid Markdown.
- Keep command handlers deterministic and local — no side effects that depend on external state.

**Rule:** Additions that require a network call or external dependency belong in a skill, not in the REPL core.

---
