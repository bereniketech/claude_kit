---
description: Invoke company-coo — the master router that delegates any task to the right department lead (engineering, marketing, content, design, product, security, AI, ops).
---

# /company — Company COO (Master Router)

Routes to the **company-coo** agent in [agents/core/company-coo.md](agents/core/company-coo.md).

## What It Does

The COO is the top-level coordinator for the entire agent suite. It detects intent and dispatches to the correct department lead — you give it any goal and it figures out which specialists to involve.

## Department Routing Map

| Domain | Department lead | Examples |
|---|---|---|
| Software / web / mobile / desktop / infra | [software-cto](agents/core/software-cto.md) | "Build an app", "Refactor backend", "Set up CI/CD" |
| AI / ML / agents / RAG / prompts | [ai-cto](agents/core/ai-cto.md) | "Build a RAG system", "Train a model", "Design an agent" |
| Content / video / blog / podcast / newsletter | [chief-content-officer](agents/content/chief-content-officer.md) | "Make a YouTube series", "Launch a newsletter" |
| Marketing / SEO / paid / growth | [chief-marketing-officer](agents/marketing/chief-marketing-officer.md) | "Plan a launch campaign", "Audit our funnel" |
| Design / UI / brand / decks | [chief-design-officer](agents/design/chief-design-officer.md) | "Design a dashboard", "Build a brand identity" |
| Product / business / sales / CS / ops / fintech / ERP | [chief-product-officer](agents/product/chief-product-officer.md) | "Write a PRD", "Set up Stripe billing", "Build an Odoo module" |
| Security / pentest / compliance / legal | [chief-security-officer](agents/security/chief-security-officer.md) | "Pentest our API", "Get SOC 2 ready", "GDPR audit" |
| Specialists (game dev, health, OCR, search, IoT, bots, CMS, RE) | individual specialist agents | "Build a Discord bot", "Search infra for…" |

## Process

1. **Intent detection** — parses the request, identifies primary domain(s)
2. **Plan composition** — single-department or multi-department workflow
3. **Delegation** — invokes department leads in the right order with the right context
4. **Coordination** — handoffs, dependencies, parallel vs sequential
5. **Synthesis** — merges deliverables into one unified output

## When to Use

- "I want to launch a new product" → triggers product + engineering + marketing + design + content
- "Take this idea from zero to live" → full-company orchestration
- "I'm not sure which agent I need" → COO will figure it out
- Any cross-functional initiative

## Inputs to Provide

- The goal (as specific or vague as you want)
- Constraints (timeline, budget, team size) — optional
- Existing context (repo, brand, audience) — optional

## Output

A unified plan with phase breakdown, agent assignments, dependencies, and a single synthesized deliverable when the workflow completes.

## Related

- All `/department-name` commands invoke a single department directly
- `/plan` — pure planning without execution
- `/orchestrate` — multi-agent execution at the engineering level
