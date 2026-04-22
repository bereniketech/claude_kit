---
name: parallel-agent-dispatch
description: Delegate 2+ independent tasks to specialized agents running concurrently. Use when facing multiple unrelated failures or independent tasks that can be worked on without shared state or sequential dependencies.
---

# Dispatching Parallel Agents

---

## 1. Overview

Delegate tasks to specialized agents with isolated context. By precisely crafting their instructions, you ensure they stay focused and succeed. Agents should never inherit your session's context or history — construct exactly what they need. This also preserves your own context for coordination work.

**Rule:** Dispatch one agent per independent problem domain. Let them work concurrently.

---

## 2. When to Use

```
Multiple failures?
  No  → Single agent investigates all
  Yes → Are they independent?
    No (related)    → Single agent investigates all
    Yes → Can they work in parallel?
      No (shared state) → Sequential agents
      Yes → Parallel dispatch
```

**Use when:**
- 3+ test files failing with different root causes
- Multiple subsystems broken independently
- Each problem can be understood without context from others
- No shared state between investigations

**Don't use when:**
- Failures are related (fix one might fix others)
- Need to understand full system state
- Agents would interfere with each other (editing same files, using same resources)
- Exploratory debugging where you don't know what's broken yet

---

## 3. The Pattern

### Identify Independent Domains

Group failures by what's broken:
- File A tests: Tool approval flow
- File B tests: Batch completion behavior
- File C tests: Abort functionality

Each domain is independent — fixing tool approval doesn't affect abort tests.

### Create Focused Agent Tasks

Each agent gets:
- **Specific scope:** One test file or subsystem
- **Clear goal:** Make these tests pass
- **Constraints:** Don't change other code
- **Expected output:** Summary of what you found and fixed

### Dispatch in Parallel

```
Agent 1 → Fix agent-tool-abort.test.ts
Agent 2 → Fix batch-completion-behavior.test.ts
Agent 3 → Fix tool-approval-race-conditions.test.ts
// All three run concurrently
```

### Review and Integrate

When agents return:
- Read each summary
- Verify fixes don't conflict
- Run full test suite
- Integrate all changes

---

## 4. Agent Prompt Structure

Good agent prompts are:
1. **Focused** — One clear problem domain
2. **Self-contained** — All context needed to understand the problem
3. **Specific about output** — What should the agent return?

Example:
```markdown
Fix the 3 failing tests in src/agents/agent-tool-abort.test.ts:

1. "should abort tool with partial output capture" - expects 'interrupted at' in message
2. "should handle mixed completed and aborted tools" - fast tool aborted instead of completed
3. "should properly track pendingToolCount" - expects 3 results but gets 0

These are timing/race condition issues. Your task:
1. Read the test file and understand what each test verifies
2. Identify root cause - timing issues or actual bugs?
3. Fix by replacing arbitrary timeouts with event-based waiting, fixing bugs if found

Do NOT just increase timeouts - find the real issue.
Return: Summary of what you found and what you fixed.
```

---

## 5. Common Mistakes

- **Too broad:** "Fix all the tests" — agent gets lost. Be specific: "Fix agent-tool-abort.test.ts"
- **No context:** "Fix the race condition" — agent doesn't know where. Paste error messages and test names.
- **No constraints:** Agent might refactor everything. Say "Do NOT change production code" or "Fix tests only."
- **Vague output:** "Fix it" — you don't know what changed. Say "Return summary of root cause and changes."

---

## 6. Verification

After agents return:
1. **Review each summary** — Understand what changed
2. **Check for conflicts** — Did agents edit same code?
3. **Run full suite** — Verify all fixes work together
4. **Spot check** — Agents can make systematic errors
