---
name: systematic-debugging
description: Structured root-cause debugging methodology. Use when encountering any bug, test failure, or unexpected behavior before proposing fixes. Four-phase process — investigate, analyze patterns, form hypotheses, then implement fixes.
---

# Systematic Debugging

---

## 1. Overview

Random fixes waste time and create new bugs. Quick patches mask underlying issues.

**Rule:** ALWAYS find root cause before attempting fixes. Symptom fixes are failure. No fixes without root cause investigation first. If you haven't completed Phase 1, you cannot propose fixes.

---

## 2. When to Use

Use for ANY technical issue: test failures, bugs in production, unexpected behavior, performance problems, build failures, integration issues.

**Use this ESPECIALLY when:**
- Under time pressure (emergencies make guessing tempting)
- "Just one quick fix" seems obvious
- You've already tried multiple fixes
- Previous fix didn't work
- You don't fully understand the issue

**Don't skip when:**
- Issue seems simple (simple bugs have root causes too)
- You're in a hurry (rushing guarantees rework)

---

## 3. Phase 1 — Root Cause Investigation

**BEFORE attempting ANY fix:**

1. **Read Error Messages Carefully**
   - Don't skip past errors or warnings — they often contain the exact solution
   - Read stack traces completely
   - Note line numbers, file paths, error codes

2. **Reproduce Consistently**
   - Can you trigger it reliably? What are the exact steps?
   - Does it happen every time?
   - If not reproducible, gather more data — don't guess

3. **Check Recent Changes**
   - What changed that could cause this? Git diff, recent commits
   - New dependencies, config changes, environmental differences

4. **Gather Evidence in Multi-Component Systems**

   When a system has multiple components (CI → build → signing, API → service → database):

   Before proposing fixes, add diagnostic instrumentation:
   ```
   For EACH component boundary:
     - Log what data enters component
     - Log what data exits component
     - Verify environment/config propagation
     - Check state at each layer

   Run once to gather evidence showing WHERE it breaks
   THEN analyze evidence to identify failing component
   THEN investigate that specific component
   ```

5. **Trace Data Flow**

   When the error is deep in a call stack, see `root-cause-tracing.md` in this directory for the complete backward tracing technique.

   Quick version: Where does the bad value originate? What called this with the bad value? Keep tracing up until you find the source. Fix at source, not at symptom.

---

## 4. Phase 2 — Pattern Analysis

**Find the pattern before fixing:**

1. **Find Working Examples** — Locate similar working code in same codebase
2. **Compare Against References** — If implementing a pattern, read reference implementation COMPLETELY. Don't skim.
3. **Identify Differences** — List every difference between working and broken, however small
4. **Understand Dependencies** — What other components, settings, config, environment does this need?

---

## 5. Phase 3 — Hypothesis and Testing

**Scientific method:**

1. **Form Single Hypothesis** — State clearly: "I think X is the root cause because Y." Be specific.
2. **Test Minimally** — Make the SMALLEST possible change to test hypothesis. One variable at a time. Don't fix multiple things at once.
3. **Verify Before Continuing** — Did it work? Yes → Phase 4. Didn't work? Form NEW hypothesis. Don't add more fixes on top.
4. **When You Don't Know** — Say "I don't understand X." Don't pretend to know. Research more.

---

## 6. Phase 4 — Implementation

**Fix the root cause, not the symptom:**

1. **Create Failing Test Case** — Simplest possible reproduction. Automated test if possible. Must have before fixing.
2. **Implement Single Fix** — Address the root cause identified. ONE change at a time. No "while I'm here" improvements. No bundled refactoring.
3. **Verify Fix** — Test passes now? No other tests broken? Issue actually resolved?
4. **If Fix Doesn't Work** — STOP. Count: How many fixes have you tried?
   - If < 3: Return to Phase 1, re-analyze with new information
   - If >= 3: STOP and question the architecture (see below)

**Rule:** If 3+ fixes have failed, STOP and question fundamentals. Pattern signs: each fix reveals new shared state/coupling, fixes require massive refactoring, each fix creates new symptoms elsewhere. This is NOT a failed hypothesis — this is a wrong architecture. Discuss with the user before attempting more fixes.

---

## 7. Red Flags — STOP and Follow Process

If you catch yourself thinking:
- "Quick fix for now, investigate later"
- "Just try changing X and see if it works"
- "Add multiple changes, run tests"
- "Skip the test, I'll manually verify"
- "It's probably X, let me fix that"
- "I don't fully understand but this might work"
- "One more fix attempt" (when already tried 2+)
- Each fix reveals new problem in different place
- Proposing solutions before tracing data flow

**ALL of these mean: STOP. Return to Phase 1.**

---

## 8. Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Issue is simple, don't need process" | Simple issues have root causes too. Process is fast for simple bugs. |
| "Emergency, no time for process" | Systematic debugging is FASTER than guess-and-check thrashing. |
| "Just try this first, then investigate" | First fix sets the pattern. Do it right from the start. |
| "I'll write test after confirming fix works" | Untested fixes don't stick. Test first proves it. |
| "Multiple fixes at once saves time" | Can't isolate what worked. Causes new bugs. |
| "Reference too long, I'll adapt the pattern" | Partial understanding guarantees bugs. Read it completely. |
| "One more fix attempt" (after 2+ failures) | 3+ failures = architectural problem. Question pattern, don't fix again. |

---

## 9. Supporting Techniques

These techniques are available in this directory:

- **`root-cause-tracing.md`** — Trace bugs backward through call stack to find original trigger
- **`defense-in-depth.md`** — Add validation at multiple layers after finding root cause
- **`condition-based-waiting.md`** — Replace arbitrary timeouts with condition polling
- **`find-polluter.sh`** — Find test pollution in test suites

---

## 10. Quick Reference

| Phase | Key Activities | Success Criteria |
|-------|---------------|------------------|
| **1. Root Cause** | Read errors, reproduce, check changes, gather evidence | Understand WHAT and WHY |
| **2. Pattern** | Find working examples, compare | Identify differences |
| **3. Hypothesis** | Form theory, test minimally | Confirmed or new hypothesis |
| **4. Implementation** | Create test, fix, verify | Bug resolved, tests pass |
