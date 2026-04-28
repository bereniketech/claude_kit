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

## 5. Resolve Required Skills

For every specialist selected in Step 4, identify the 1–4 skills from the library that agent will need. Use the agent→skill mapping table below as a starting point, then add goal-specific skills derived from the idea (tech stack, framework, integration, domain).

### Agent → Primary Skill Mapping

| Agent | Primary skills |
|---|---|
| `@architect` | `architecture/software-architecture` · `architecture/microservices-patterns` · `architecture/ddd-*` |
| `@planner` | `planning/planning-specification-architecture-software` · `planning/autonomous-agents-task-automation` |
| `@software-developer-expert` | `development/code-writing-software-development` · `development/systematic-debugging` |
| `@web-frontend-expert` | `frameworks-frontend/react-*` or relevant FE framework · `ui-design/ui-ux-pro-max` |
| `@web-backend-expert` | `frameworks-backend/` relevant framework · `data-backend/postgres-patterns` |
| `@mobile-expert` | `frameworks-mobile/flutter-expert` or `react-native-*` or `ios-*` |
| `@desktop-expert` | `frameworks-desktop/electron-development` or relevant desktop skill |
| `@systems-programming-expert` | `languages/rust-pro` · `languages/cpp-patterns` · `os-linux-platform/linux-kernel-config-for-custom-os` |
| `@mcp-server-expert` | `ai-platform/claude-developer-platform` |
| `@python-expert` | `languages/python-patterns` |
| `@typescript-expert` | `languages/typescript-*` |
| `@polyglot-expert` | language skill matching the stack in the idea |
| `@code-reviewer` | `testing-quality/security-review` · `development/code-writing-software-development` |
| `@refactor-cleaner` | `development/code-writing-software-development` |
| `@doc-updater` | `research-docs/document-content-writing-editing` |
| `@build-error-resolver` | `devops/terminal-cli-devops` |
| `@ai-ml-expert` | `data-science-ml/ai-engineer` · `data-science-ml/ml-engineer` |
| `@ai-platform-expert` | `ai-platform/claude-developer-platform` · `data-science-ml/llm-app-patterns` |
| `@orchestration-expert` | `agents-orchestration/agent-framework-*` · `agents-orchestration/conductor-*` |
| `@data-scientist-expert` | `data-science-ml/rag-*` · `data-science-ml/hugging-face-*` |
| `@devops-infra-expert` | `devops/terminal-cli-devops` · `containers-orchestration/docker-expert` · `containers-orchestration/kubernetes-*` |
| `@cloud-architect` | `cloud/aws-serverless` or `cloud/gcp-cloud-run` or `cloud/terraform-*` |
| `@azure-expert` | `cloud-azure/` relevant Azure service skills |
| `@observability-engineer` | `observability/distributed-tracing` · `observability/prometheus-configuration` |
| `@database-architect` | `data-backend/postgres-patterns` · `data-backend/database-migrations` · `data-backend/vector-database-engineer` |
| `@database-reviewer` | `data-backend/database-admin` |
| `@test-expert` | `testing-quality/tdd-workflow` · `testing-quality/e2e-testing` |
| `@tdd-guide` | `testing-quality/tdd-workflow` |
| `@e2e-runner` | `testing-quality/e2e-testing` · `testing-quality/playwright-*` |
| `@security-reviewer` | `testing-quality/security-review` · `security-defensive/semgrep-*` |
| `@pentest-expert` | `security-offensive/pentest-*` · `security-offensive/burpsuite-*` |
| `@security-architect` | `security-defensive/threat-modeling-*` · `security-defensive/auth-implementation-patterns` |
| `@legal-compliance-expert` | `legal-compliance/gdpr-data-handling` · `legal-compliance/legal-advisor` |
| `@ui-design-expert` | `ui-design/ui-ux-pro-max` · `ui-design/design-system` · `ui-design/accessibility-*` |
| `@product-manager-expert` | `product-business/product-manager` · `product-business/saas-*` |
| `@startup-analyst` | `product-business/startup-*` · `product-business/analytics-*` |
| `@ecommerce-expert` | `ecommerce/shopify-*` or `ecommerce/wordpress-woocommerce` |
| `@fintech-payments-expert` | `fintech-payments/stripe-integration` · `fintech-payments/plaid-fintech` |
| `@saas-integrations-expert` | `integrations/` relevant SaaS skills |
| `@workflow-automation-expert` | `integrations/zapier-*` · `integrations/n8n-*` |
| `@game-dev-expert` | `game-dev/unity-*` or `game-dev/godot-*` or `game-dev/unreal-engine-cpp-pro` |
| `@cms-expert` | `cms/wordpress` · `cms/wordpress-plugin` |
| `@seo-expert` | `seo/seo-technical` · `seo/seo-audit` · `seo/seo-content-*` |
| `@growth-marketing-expert` | `marketing-growth/growth-engine` · `marketing-growth/viral-generator-builder` |
| `@paid-ads-expert` | `marketing-growth/paid-ads` |
| `@email-marketing-expert` | `marketing-growth/cold-email` |
| `@brand-expert` | `marketing-growth/content-strategy` · `ui-design/design-system` |
| `@blog-writing-expert` | `research-docs/blog-writing-guide` · `research-docs/document-content-writing-editing` |
| `@youtube-content-expert` | `media-video/remotion-*` |
| `@podcast-expert` | `media-video/podcast-generation` |
| `@technical-writer-expert` | `research-docs/document-content-writing-editing` · `research-docs/wiki-*` |
| `@social-media-expert` | `marketing-growth/content-strategy` |
| `@os-userland-architect` | `os-linux-platform/linux-kernel-config-for-custom-os` · `os-linux-platform/drm-kms-and-mesa-basics` |
| `@linux-platform-expert` | `os-linux-platform/hardware-profiling-and-driver-mapping` · `os-linux-platform/pipewire-wireplumber-session` · `os-linux-platform/input-stack-libinput-udev` |

**Additional skill resolution rules:**

| Signal in the idea | Add these skills |
|---|---|
| Specific language mentioned (Go, Rust, Python, etc.) | `languages/<lang>-patterns` or `languages/<lang>-pro` |
| Specific framework mentioned (React, Next.js, Django, etc.) | `frameworks-<tier>/<framework>-*` |
| Cloud provider mentioned | `cloud/<provider>-*` or `cloud-azure/<service>` |
| Auth / login / OAuth | `security-defensive/auth-implementation-patterns` |
| Real-time / WebSocket | `frameworks-backend/nodejs-*` · `data-backend/event-sourcing-architect` |
| RAG / embeddings / vector search | `data-science-ml/rag-*` · `data-backend/vector-database-engineer` |
| Payments | `fintech-payments/stripe-integration` |
| CI/CD | `devops/github-actions-*` or `devops/gitlab-ci-*` |
| Scraping / crawling | `integrations-scraping/firecrawl-scraper` or `integrations-scraping/apify-*` |
| Accessibility | `ui-design/accessibility-*` |
| Performance | `performance/web-performance` or `performance/app-performance` |

Deduplicate. Cap at 8 skills total per call chain. Prefer specificity over breadth.

Skill import path format: `@C:/Users/Hp/Desktop/Experiment/claude_kit/skills/<category>/<skill-name>/SKILL.md`

---

## 6. Output the Call Chain

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

### Skills Required
Add these to your project's .claude/CLAUDE.md before execution:

@C:/Users/Hp/Desktop/Experiment/claude_kit/skills/<category>/<skill-name>/SKILL.md
@C:/Users/Hp/Desktop/Experiment/claude_kit/skills/<category>/<skill-name>/SKILL.md
...

### Hierarchy Rule
ALWAYS follow: board → CEO → [sub-lead if applicable] → specialist
NEVER skip a level. NEVER invoke specialists directly.
```

If the idea spans multiple companies, repeat the specialist block and skills block for each company, then deduplicate the combined skills list at the end.

---

## 7. Provide Ready-to-Use Brief

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
