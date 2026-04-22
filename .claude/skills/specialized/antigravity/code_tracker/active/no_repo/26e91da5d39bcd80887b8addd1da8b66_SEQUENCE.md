�*# 🧠 Core Strategy

### Golden Rule:

Claude should never hold more than 3–4 modules worth of complexity at once.

We break the system into **5 controlled phases**.

Each phase:

* Builds a layer
* Locks it
* Tests it
* Moves forward

Never move forward with untested foundation.

---

# 🧱 PHASE 1 — FOUNDATION LAYER

### Modules to give together:

* MODULE 1 — Project Skeleton
* MODULE 3 — Config + Env
* MODULE 16 — Constants
* CLAUDE.md (final version)

Do NOT give database yet.

### Why?

This builds:

* App boot
* Env validation
* Constants enforcement
* Folder structure
* Strict discipline rules

### Prompt Structure for Phase 1

Paste:

1. CLAUDE.md
2. Then say:

> Implement Modules 1, 3, and 16 only.
> Do not implement any business logic.
> Ensure startup fails if env missing.
> Add complete comments.

Keep it focused.

---

# 🗄 PHASE 2 — DATABASE LAYER

### Modules to give together:

* MODULE 2 — Full SQL schema
* database.py
* SQLAlchemy models
* Soft delete rules
* Unique constraints explanation

Nothing else.

### Why separate?

Schema is large.
Mixing with services will cause hallucinated fields.

### Prompt Structure

> Implement database layer only.
> Use exact schema provided.
> Do not add extra columns.
> Add full docstrings explaining constraints.
> Do not implement services yet.

After completion:

Review models carefully.

If schema drifts → correct immediately before continuing.

---

# 🔐 PHASE 3 — SECURITY + INFRASTRUCTURE

### Modules to give together:

* MODULE 4 — Webhook layer
* Signature validation requirement
* MODULE 15 — Admin auth validation
* Retry utilities (MODULE 17)
* Date utilities (MODULE 18)

### Why group these?

They are infrastructure utilities:

* Parsing
* Validation
* Retry
* Security

No business logic yet.

### Important



> Do not implement onboarding or preventive logic.
> Only build routing, parsing, retry, security utilities.

Keep context small.

---

# 🐾 PHASE 4 — CORE BUSINESS ENGINE

This is the largest phase.

Break it into 2 sub-phases.

---

## PHASE 4A — Preventive Logic Engine

Give together:

* MODULE 6 — Preventive Seeder
* MODULE 9 — Preventive Calculator
* MODULE 8 — Conflict Engine
* MODULE 19 — Conflict Expiry

Do NOT include reminder engine yet.

Why?

Preventive calculation must be stable before reminders use it.

---

## PHASE 4B — Reminder System

Give together:

* MODULE 10 — Reminder Engine
* MODULE 11 — Reminder Response State Machine
* Rate limit rule
* Deduplication rule

Keep extraction separate.

---

# 📄 PHASE 5 — DOCUMENT + AI + DASHBOARD

Break into two parts.

---

## PHASE 5A — Document & Extraction

Give together:

* MODULE 7 — Upload + GPT Extraction
* OpenAI rules
* MIME validation
* Daily upload limit

Do not include query engine yet.

---

## PHASE 5B — Dashboard + Query Engine

Give together:

* MODULE 13 — Dashboard
* MODULE 12 — Health Score
* MODULE 14 — Query Engine

This keeps AI contexts separate (extraction vs query).

---

# 🧠 TOKEN-SAFE INFORMATION DELIVERY METHOD

Instead of pasting full module text every time:

Use this pattern:

### Step 1

Paste only the specific module instructions needed.

### Step 2

After implementation, say:

> Confirm no hardcoded secrets, no recurrence hardcoding, and strict docstrings included.

This keeps Claude focused.

---

# 🚨 CRITICAL RULE TO AVOID DRIFT

At the start of each new phase, paste this reminder:


> Follow CLAUDE.md strictly.
> Do not modify schema.
> Do not invent new fields.
> Do not simplify constraints.

This resets guardrails.

---

# 🪜 EXACT EXECUTION ORDER

1. Phase 1 — Foundation
2. Phase 2 — Database
3. Phase 3 — Infrastructure + Security
4. Phase 4A — Preventive Engine
5. Phase 4B — Reminder System
6. Phase 5A — Extraction
7. Phase 5B — Dashboard + Query

Do not change order.

---

# 🧩 WHY THIS WORKS

Because:

* Schema defined before logic
* Logic defined before automation
* Automation defined before AI
* AI defined before dashboard
* Each layer depends only on previous stable layer

This minimizes token reuse.

---

# 📦 What NOT To Do

❌ Do not paste all 19 modules at once
❌ Do not mix extraction + reminder + onboarding
❌ Do not ask Claude to "build full system"
❌ Do not re-send schema repeatedly

---

# 🎯 Advanced Token Optimization Trick

After each phase is complete:

Ask Claude:

> Summarize implemented components in 15 lines for internal reference.

Then use that summary instead of re-sending full code in later prompts.

This dramatically reduces context load.

---

# 🏗 Estimated Safe Prompt Size Strategy

| Phase    | Estimated Safe Token Range |
| -------- | -------------------------- |
| Phase 1  | 3k–4k                      |
| Phase 2  | 4k                         |
| Phase 3  | 3k                         |
| Phase 4A | 4k                         |
| Phase 4B | 4k                         |
| Phase 5A | 4k                         |
| Phase 5B | 4k                         |

Never exceed ~5k tokens per instruction batch.


� ����
�� ��
�� ��
�� ��
��* 2<file:///c:/Users/Hp/Desktop/Experiment/Pet%20MVP/SEQUENCE.md