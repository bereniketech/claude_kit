---
name: eval-harness
description: Apply eval-driven development (EDD) by defining pass/fail criteria before implementation and measuring reliability with pass@k metrics.
---

# Eval Harness

Define expected behavior before writing code. Treat evals as the unit tests of AI-assisted development.

---

## 1. Eval-Before-Code Philosophy

Write eval definitions before any implementation. Evals force precise thinking about success criteria and create a measurable definition of done. Store eval definitions as version-controlled artifacts alongside the code they govern.

**Rule:** Never start implementation on a new feature or agent change without a written eval definition file first.

---

## 2. Capability Evals

Use capability evals to verify Claude can accomplish something new. Define the task, enumerate success criteria as checkboxes, and describe the expected output in observable terms.

```markdown
[CAPABILITY EVAL: feature-name]
Task: Description of what the agent must accomplish
Success Criteria:
  - [ ] Criterion 1
  - [ ] Criterion 2
  - [ ] Criterion 3
Expected Output: Description of expected result
```

**Rule:** Every success criterion must be independently verifiable — no compound criteria in a single checkbox.

---

## 3. Regression Evals

Use regression evals to ensure existing functionality remains intact after a change. Reference a baseline commit or checkpoint. Record pass/fail for every test in the set.

```markdown
[REGRESSION EVAL: feature-name]
Baseline: <git SHA or checkpoint name>
Tests:
  - existing-test-1: PASS/FAIL
  - existing-test-2: PASS/FAIL
Result: X/Y passed (previously Y/Y)
```

**Rule:** A regression eval that drops below its prior score is a blocking finding — do not ship.

---

## 4. pass@k and pass^k Metrics

Use `pass@k` ("at least one success in k attempts") to measure practical reliability. Use `pass^k` ("all k attempts succeed") to measure stability on critical paths. Track both metrics over time to detect reliability drift.

- `pass@1`: first-attempt success rate
- `pass@3`: success within 3 attempts — target ≥ 90% for capability evals
- `pass^3`: all 3 consecutive runs pass — require 100% for release-critical regression evals

**Rule:** Never promote a feature to production with `pass@3 < 0.90` on its capability evals.

---

## 5. Grader Types

Choose the grader type that provides the most determinism for each criterion.

- **Code grader**: deterministic shell assertions (`grep -q`, `npm test`, `npm run build`)
- **Model grader**: LLM-as-judge rubric scoring 1–5 with explicit reasoning — use for open-ended output quality
- **Human grader**: flag for manual review with a stated risk level — required for security and financial logic

**Rule:** Default to code graders. Use model graders only when deterministic checks are impossible. Always use human graders for security-relevant changes.

---

## 6. Eval Workflow

Follow four phases for every feature or agent change.

1. **Define** (before coding): write the eval definition file at `.claude/evals/<feature>.md`
2. **Implement**: write code targeting the defined criteria
3. **Evaluate**: run each criterion, record PASS/FAIL and attempt count
4. **Report**: produce a structured summary with pass@k scores and a ship/block decision

```markdown
EVAL REPORT: feature-xyz
Capability: 3/3 passed — pass@1: 67%, pass@3: 100%
Regression: 3/3 passed — pass^3: 100%
Status: READY FOR REVIEW
```

**Rule:** An eval report is a required artifact for every PR that modifies agent behavior or prompts.

---

## 7. Eval Storage Layout

Store all eval artifacts under `.claude/evals/` so they are versioned with the code.

```
.claude/
  evals/
    <feature>.md       # eval definition
    <feature>.log      # run history
    baseline.json      # regression baselines
docs/releases/<ver>/
    eval-summary.md    # release snapshot
```

**Rule:** Never delete eval logs — they are the longitudinal record of agent reliability.

---

## 8. Eval Anti-Patterns

Do not overfit prompts to known eval examples. Do not measure only happy-path outputs. Do not ignore latency and cost drift while optimizing pass rates. Do not allow flaky graders in release gates — a grader that produces inconsistent results on the same input must be fixed or replaced before it gates any release.

**Rule:** If a grader itself is flaky, the eval suite is invalid — fix the grader before running any evaluation.
