---
name: ralphinho-rfc-pipeline
description: RFC-driven multi-agent DAG execution with quality gates, merge queues, and work unit orchestration for features too large for a single agent pass.
---

# Ralphinho RFC Pipeline

Use this skill when a feature is too large for a single agent pass and must be split into independently verifiable work units. Applies RFC-driven decomposition into a DAG of units, each with its own acceptance tests, risk tier, and quality pipeline.

> Note: The orchestration patterns from this skill have been merged into `autonomous-agents-task-automation`. Refer to that skill for broader multi-agent delegation and parallel execution guidance.

---

## 1. Pipeline Stages

Execute these stages in order:

1. RFC intake — capture requirements, constraints, and success criteria
2. DAG decomposition — break the RFC into a directed acyclic graph of work units
3. Unit assignment — assign units to agents based on scope and risk tier
4. Unit implementation — implement each unit per its spec
5. Unit validation — run acceptance tests for the unit in isolation
6. Merge queue — queue validated units for integration in dependency order
7. Final system verification — run full integration tests after all merges

---

## 2. Work Unit Spec

Each work unit must include:

| Field | Purpose |
|-------|---------|
| `id` | Unique identifier |
| `depends_on` | IDs of upstream units that must complete first |
| `scope` | Files and behaviors this unit touches |
| `acceptance_tests` | Pass/fail criteria that define done |
| `risk_level` | Complexity tier (see below) |
| `rollback_plan` | How to revert if this unit fails integration |

---

## 3. Complexity Tiers

Assign each unit a tier before implementation begins:

| Tier | Characteristics |
|------|----------------|
| 1 | Isolated file edits, deterministic tests, no cross-service impact |
| 2 | Multi-file behavior changes, moderate integration risk |
| 3 | Schema, auth, performance, or security changes — highest scrutiny |

**Rule:** Tier 3 units require a rollback plan before implementation starts, not after.

---

## 4. Quality Pipeline per Unit

Each unit passes through this sequence before entering the merge queue:

1. Research — confirm approach, check for prior art in codebase
2. Implementation plan — spec the changes before writing code
3. Implementation — write the code
4. Tests — write and run acceptance tests
5. Review — verify against the unit spec
6. Merge-ready report — document what changed, test results, and any residual risk

---

## 5. Merge Queue Rules

- Never merge a unit with unresolved dependency failures.
- Always rebase unit branches on the latest integration branch before merging.
- Re-run integration tests after each queued merge — not just at the end.

**Rule:** A unit that passes in isolation but breaks integration must be evicted and its scope re-examined.

---

## 6. Recovery

If a unit stalls or blocks the queue:

1. Evict from the active queue.
2. Snapshot findings and partial work.
3. Regenerate a narrowed unit scope that unblocks progress.
4. Retry with updated constraints.

---

## 7. Outputs

Produce these artifacts at pipeline completion:

- RFC execution log
- Per-unit scorecards (scope, test results, risk)
- Dependency graph snapshot
- Integration risk summary

---
