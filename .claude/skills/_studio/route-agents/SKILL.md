---
name: route-agents
description: Understand an idea and produce a precise agent call chain — which agents to invoke, in what order, and what to pass each one. Analysis and routing only; never plans or executes.
---

# Route Agents

Takes any idea or goal and maps it to the correct agent hierarchy: board entry point → company CEO → specialists. Outputs a ready-to-use call chain with briefs for each hop.

---

## 1. Gather the Idea

If the user has not described their goal, ask exactly one question:

> "What are you trying to accomplish? (e.g. build a feature, launch a campaign, produce content, hire someone, fix a security issue)"

Do not ask follow-up questions. Derive everything from the answer.

---

## 2. Classify the Goal

Map the idea to one or more companies using this table:

| Goal type | Company | Board entry |
|---|---|---|
| Software, product, AI, infra, APIs, data, security, OS | software-company | `@company-coo` |
| SEO, ads, growth, brand, email campaigns | marketing-company | `@company-coo` |
| Blog, video, podcast, social, newsletter, docs | media-company | `@company-coo` |
| Spans multiple companies | multi-company | `@company-coo` |
| Design coherence across UI + brand + visuals | cross-company design | `@chief-design-officer` |
| Hiring, comp, contracts, handbook, onboarding | HR / people ops | `@people-operations-expert` |
| Comms triage, decision tracking, escalations | ops / comms | `@chief-of-staff` |

**Rule:** When in doubt, start with `@company-coo`. The COO routes to board peers when needed.

---

## 3. Map the Full Agent Chain

For each company involved, trace the complete routing path from board to specialists.

### software-company routing

```
@company-coo
  └── @software-cto
        ├── engineering/
        │   ├── @architect              — new modules, data model changes, external integrations
        │   ├── @planner                — task breakdown and sequencing
        │   ├── @software-developer-expert
        │   ├── @web-frontend-expert
        │   ├── @web-backend-expert
        │   ├── @mobile-expert
        │   ├── @desktop-expert
        │   ├── @systems-programming-expert
        │   ├── @mcp-server-expert
        │   ├── @python-expert
        │   ├── @typescript-expert
        │   ├── @polyglot-expert
        │   ├── @cinematic-website-builder
        │   ├── @code-reviewer
        │   ├── @refactor-cleaner
        │   ├── @doc-updater
        │   └── @build-error-resolver
        ├── ai/
        │   ├── @ai-cto               — sub-lead for all AI/ML work
        │   ├── @ai-ml-expert
        │   ├── @ai-platform-expert
        │   ├── @orchestration-expert
        │   └── @data-scientist-expert
        ├── devops/
        │   ├── @devops-infra-expert
        │   ├── @cloud-architect
        │   ├── @azure-expert
        │   └── @observability-engineer
        ├── data/
        │   ├── @database-architect
        │   └── @database-reviewer
        ├── qa/
        │   ├── @test-expert
        │   ├── @tdd-guide
        │   ├── @e2e-runner
        │   └── @security-reviewer
        ├── languages/
        │   ├── @go-reviewer / @go-build-resolver
        │   ├── @kotlin-reviewer / @kotlin-build-resolver
        │   └── @python-reviewer
        ├── product/
        │   ├── @chief-product-officer — sub-lead for product/business work
        │   ├── @product-manager-expert
        │   ├── @ecommerce-expert
        │   ├── @startup-analyst
        │   ├── @customer-success-expert
        │   ├── @sales-automation-expert
        │   ├── @saas-integrations-expert
        │   ├── @workflow-automation-expert
        │   ├── @erp-odoo-expert
        │   └── @fintech-payments-expert
        ├── design/
        │   └── @ui-design-expert
        ├── security/
        │   ├── @chief-security-officer — sub-lead for security/compliance
        │   ├── @pentest-expert
        │   ├── @security-architect
        │   └── @legal-compliance-expert
        ├── specialists/
        │   ├── @game-dev-expert
        │   ├── @office-automation-expert
        │   ├── @search-expert
        │   ├── @enterprise-operations-expert
        │   ├── @conversational-agent-expert
        │   ├── @cms-expert
        │   └── @reverse-engineering-expert
        └── os-engineering/
            ├── @os-userland-architect — custom OS / desktop runtime / agentic-OS
            └── @linux-platform-expert
```

### marketing-company routing

```
@company-coo
  └── @chief-marketing-officer
        ├── strategy/
        │   ├── @seo-expert
        │   ├── @growth-marketing-expert
        │   └── @competitor-intelligence-expert
        ├── campaigns/
        │   ├── @paid-ads-expert
        │   └── @email-marketing-expert
        └── brand/
            └── @brand-expert
```

### media-company routing

```
@company-coo
  └── @chief-content-officer
        ├── video/
        │   ├── @youtube-content-expert
        │   └── @video-production-expert
        ├── audio/
        │   └── @podcast-expert
        ├── editorial/
        │   ├── @blog-writing-expert
        │   ├── @newsletter-expert
        │   └── @technical-writer-expert
        ├── visual/
        │   ├── @image-creation-expert
        │   └── @presentation-expert
        └── distribution/
            ├── @social-media-expert
            └── @content-repurposing-expert
```

---

## 4. Select Agents for This Idea

From the classified goal (Step 2) and the full tree (Step 3), identify:

1. **Board entry point** — always one agent
2. **Company CEO** — the CEO who owns the work
3. **Sub-leads** (if applicable) — `@ai-cto`, `@chief-product-officer`, `@chief-security-officer`, `@os-userland-architect`
4. **Specialists** — the leaf agents who will actually do the work

**Selection rules:**

| Signal in the idea | Add this agent |
|---|---|
| New module, service boundary, or data model change | `@architect` before any developer |
| Task sequencing and breakdown | `@planner` |
| Any testing required | `@test-expert` or `@tdd-guide` |
| Deployment / infra | `@devops-infra-expert` or `@cloud-architect` |
| Security audit needed | `@security-reviewer` + `@security-architect` |
| AI / ML feature | Route through `@ai-cto` first |
| Database design | `@database-architect` before `@web-backend-expert` |
| Code quality after build | `@code-reviewer` → `@refactor-cleaner` |

---

## 5. Output the Call Chain

Print the chain in this exact format — one block per company, ordered from board to leaf:

```
## Agent Call Chain for: [idea summary]

### Entry Point
Invoke: @[board-agent]
Brief: [1–2 sentence summary of what you're handing off]

### [Company Name] — CEO: @[ceo-agent]
Route reason: [why this company owns the work]

### Specialists (in order)
1. @[agent-name] — [one-line reason why this agent is needed]
2. @[agent-name] — [one-line reason]
3. @[agent-name] — [one-line reason]
...

### Hierarchy Rule
ALWAYS follow: board → CEO → [sub-lead if applicable] → specialist
NEVER skip a level. NEVER invoke specialists directly.
```

If the idea spans multiple companies, repeat the specialist block for each company.

---

## 6. Provide Ready-to-Use Brief

After the call chain, output a copy-paste brief the user can send to the board agent:

```
---
**Paste this to @[board-agent]:**

[Board agent], I need your help with the following:

**Goal:** [idea in 1–2 sentences]
**Scope:** [what this touches — software / marketing / media / cross-company]
**Constraints:** [any known deadlines, tech stack, or boundaries]
**Expected output:** [requirements + design + tasks, or deliverable type]

Please assess, assign to the right company CEO, and follow the full delegation chain through to specialists. Each specialist must run the planning skill before any execution begins, and I must approve requirements → design → tasks before work starts.
```

---

## Rules

- **Analysis + routing only.** Never plan, never write code, never create tasks.
- **Never skip a level.** Board → CEO → specialist is non-negotiable.
- **Never recommend invoking a specialist directly.** Always route through the board first.
- **One entry point per invocation.** If multi-company, still start with `@company-coo` — the COO coordinates between CEOs.
- **Plans come from specialists.** Board members and CEOs never produce plans themselves.
