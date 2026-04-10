# Agent Suite Build Plan — Full Company OS

**Goal:** Transform claude_kit's 1,200+ skills into a hierarchical agent suite that can autonomously run a full software, media, and marketing company.

**Reference:** `BRAIN.md` | **Agent format:** `agents/development/code-reviewer.md`

---

## ✅ Phase 0 — Foundational Cleanup (COMPLETE)

### Tier 1 Skill Removals (done)
| Removed | Reason |
|---|---|
| `skills/security-offensive/ffuf-claude-skill/` | 23-line stub, superseded by `ffuf-web-fuzzing` |
| `skills/security-defensive/incident-response-incident-response/` | Malformed double-paste name |
| `skills/data-science-ml/llm-application-dev-prompt-optimize/` | 40-line truncated stub |
| `skills/seo/seo/` | Orchestrator replaced by `seo-expert` agent |

### `skills/_studio/` Created (done)
Entry-point skills moved from `skills/core/` to `skills/_studio/`:
- `skills/_studio/generate-claude-md/SKILL.md` — updated with agent routing (Step 3b + Active Agent Team)
- `skills/_studio/new-features-updates/SKILL.md` — updated with Step 4b agent routing

### Existing Agents Upgraded (done)
| Agent | Key upgrade |
|---|---|
| `database-reviewer` | Full database architect (PostgreSQL, NoSQL, ClickHouse, vector, CQRS, migrations) |
| `tdd-guide` | Full TDD suite (Red-Green-Refactor, Playwright E2E, k6 perf, eval-driven, all frameworks) |
| `security-reviewer` | Full OWASP Top 10 + secrets management + K8s + GDPR |
| `e2e-runner` | Full Playwright suite with POM, mocking, CI, flaky handling |
| `python-reviewer` | Full Python review (security, typing, async, Django/FastAPI/Flask) |
| `go-reviewer` | Added inline Go patterns reference |
| `go-build-resolver` | Added module patterns |
| `kotlin-reviewer` | Already comprehensive — kept as-is |
| `kotlin-build-resolver` | Added KMP patterns |
| `planner` | Added department routing context |
| `refactor-cleaner` | Added FP refactoring patterns |
| `doc-updater` | Added technical writing standards |
| `build-error-resolver` | Added multi-language build patterns |
| `loop-operator` | Full loop management (patterns, monitoring, stall detection, recovery) |
| `harness-optimizer` | Full audit system (hooks, evals, routing, context, safety) |
| `chief-of-staff` | Added company routing context |
| `architect` | Full architecture expert (DDD, CQRS, event-driven, API design, ADRs, cloud-native) |

---

## ✅ Phase 1 — New Skills Created (COMPLETE)

16 new skills in WAT format:

| Skill | Path |
|---|---|
| YouTube Content Creator | `skills/media-video/youtube-content-creator/` |
| YouTube Transcript | `skills/media-video/youtube-transcript/` |
| Video Script Writing | `skills/media-video/video-script-writing/` |
| Thumbnail Design System | `skills/ui-design/thumbnail-design-system/` |
| AI Video Generation | `skills/media-video/ai-video-generation/` |
| Newsletter Creation | `skills/research-docs/newsletter-creation/` |
| Email Marketing Automation | `skills/marketing-growth/email-marketing-automation/` |
| Social Media Management | `skills/marketing-growth/social-media-management/` |
| Content Repurposing | `skills/media-video/content-repurposing/` |
| MCP Server Development | `skills/ai-platform/mcp-server-development/` |
| Competitor Intelligence | `skills/marketing-growth/competitor-intelligence/` |
| Sales Automation | `skills/product-business/sales-automation/` |
| Customer Success | `skills/product-business/customer-success/` |
| Deep Research | `skills/research-docs/deep-research/` |
| Resume Tailoring | `skills/team-hr/resume-tailoring/` |
| Invoice Organizer | `skills/office-documents/invoice-organizer/` |

---

## ✅ Phase 2 — Content & Media Agents (COMPLETE)

Create these agents in `agents/content/`:

| Agent | Key skills to embed | MCP |
|---|---|---|
| `youtube-content-expert.md` | youtube-content-creator, video-script-writing, thumbnail-design-system, youtube-transcript, content-repurposing | exa-web-search, browser-use, fal-ai |
| `video-production-expert.md` | video-script-writing, ai-video-generation, media-video/* | fal-ai |
| `image-creation-expert.md` | ui-design/imagen, stability-ai, fal-ai-media, thumbnail-design-system | fal-ai, magic |
| `blog-writing-expert.md` | blog-writing-guide, research-docs/*, document-writing, seo-content | exa-web-search, context7 |
| `social-media-expert.md` | social-media-management, content-repurposing, marketing-growth/* | browser-use |
| `podcast-expert.md` | podcast-generation, audio-transcriber, newsletter-creation | fal-ai |
| `newsletter-expert.md` | newsletter-creation, email-marketing-automation, content-repurposing | |
| `content-repurposing-expert.md` | content-repurposing, social-media-management, all content formats | |
| `technical-writer-expert.md` | document-writing, wiki/*, research-docs/*, changelog-automation | context7, github |
| `chief-content-officer.md` | Coordinates all above agents | |

---

## ✅ Phase 3 — Marketing Agents (COMPLETE)

Created in `agents/marketing/`:

| Agent | Key skills embedded | MCP |
|---|---|---|
| `seo-expert.md` | seo/* (all 31 skills), programmatic-seo, deep-research | exa-web-search, firecrawl, context7 |
| `growth-marketing-expert.md` | marketing-growth/* (31 skills), growth-engine, viral-generator | exa-web-search, browser-use |
| `paid-ads-expert.md` | paid-ads, marketing-growth/paid-ads, competitor-intelligence | browser-use, firecrawl |
| `email-marketing-expert.md` | email-marketing-automation, newsletter-creation, marketing-growth/* | — |
| `competitor-intelligence-expert.md` | competitor-intelligence, deep-research, integrations-scraping/* | firecrawl, exa-web-search, browser-use |
| `chief-marketing-officer.md` | Coordinates all above agents | — |

---

## ✅ Phase 4 — Software Engineering Agents (COMPLETE)

Created in `agents/development/`, `agents/devops/` (new), `agents/data-backend/`, `agents/testing-quality/`, and `agents/core/`:

| Agent | Path | Key domain |
|---|---|---|
| `software-developer-expert.md` | `agents/development/` | Generalist coding, debugging, API design, branch completion, framework migration |
| `web-frontend-expert.md` | `agents/development/` | React, Next.js, Angular, Svelte, Tailwind, animations, a11y, performance |
| `web-backend-expert.md` | `agents/development/` | FastAPI, Django, NestJS, Laravel, Express, Prisma, queues, auth, API design |
| `mobile-expert.md` | `agents/development/` | iOS (Swift), Android (Kotlin), Flutter, React Native, Expo, KMP |
| `desktop-expert.md` | `agents/development/` | Electron, Tauri, Avalonia, native, PWA, distribution & auto-update |
| `mcp-server-expert.md` | `agents/development/` | MCP server design, tools, resources, prompts, transports, distribution |
| `devops-infra-expert.md` | `agents/devops/` (new) | Docker, Kubernetes, CI/CD, GitOps, service mesh, deployment strategies |
| `cloud-architect.md` | `agents/devops/` | AWS, GCP, Cloudflare, Vercel, Hetzner, Terraform, multi-region, FinOps |
| `azure-expert.md` | `agents/devops/` | Full Azure stack — App Service, Functions, AKS, Cosmos, Entra ID, Bicep |
| `observability-engineer.md` | `agents/devops/` | OTel tracing, Prometheus metrics, logs, SLOs, alerting, incident response |
| `database-architect.md` | `agents/data-backend/` | Greenfield schema design, sharding, CQRS, event sourcing, vector DBs |
| `systems-programming-expert.md` | `agents/development/` | Rust, C, C++, memory, concurrency, FFI, embedded, perf, gdb/perf/strace |
| `python-expert.md` | `agents/development/` | Modern Python, typing, async, Django, FastAPI, SQLAlchemy, uv, ML/AI |
| `typescript-expert.md` | `agents/development/` | TS type system mastery, zod, Effect, monorepo, library authoring |
| `polyglot-expert.md` | `agents/development/` | Go, Java, Kotlin, Swift, C#, Scala, Ruby, PHP, Elixir, Haskell, Clojure |
| `test-expert.md` | `agents/testing-quality/` | Strategy, unit/integration/e2e, perf (k6), a11y, visual, fuzz, eval-driven |
| `software-cto.md` | `agents/core/` | Master coordinator routing all software engineering work to specialists |

---

## ✅ Phase 5 — AI/ML, Design, Product, Security Agents (COMPLETE)

### AI/ML (`agents/ai/`)
| Agent | Path | Key domain |
|---|---|---|
| `ai-ml-expert.md` | `agents/ai/` | Prompt eng, RAG, embeddings, vector DBs, eval, multimodal/voice/CV, ML eng, LangChain/Graph/LlamaIndex/DSPy |
| `ai-platform-expert.md` | `agents/ai/` | Claude API, Agent SDK, MCP servers, prompt caching, computer use, claude-d3js, NotebookLM |
| `orchestration-expert.md` | `agents/ai/` | Multi-agent topologies, agent memory, conductor, multi-advisor, eval, tool-use guardian, computer-use agents |
| `data-scientist-expert.md` | `agents/ai/` | EDA, pandas/polars, stats, experiments, viz, classical ML, scientific computing (astropy/biopython/qiskit) |

### Design (`agents/design/`)
| Agent | Path | Key domain |
|---|---|---|
| `ui-design-expert.md` | `agents/design/` | Design systems, components, a11y (WCAG 2.2), responsive, dashboards, forms, dark mode, motion, HIG, Material |
| `brand-expert.md` | `agents/design/` | Brand strategy, naming, logo, voice/tone, visual identity, guidelines, rebranding, brand architecture |
| `presentation-expert.md` | `agents/design/` | Pitch decks, conference talks, exec presentations, sales decks, story structure, data viz, delivery |
| `chief-design-officer.md` | `agents/design/` | Coordinates design agents — strategy, ops, hiring, multi-agent campaigns |

### Product & Business (`agents/product/`)
| Agent | Path | Key domain |
|---|---|---|
| `product-manager-expert.md` | `agents/product/` | Strategy, roadmaps, OKRs, JTBD, RICE/Kano/MoSCoW/WSJF, PRDs, launches, experimentation, retention |
| `ecommerce-expert.md` | `agents/product/` | Shopify (Liquid, Hydrogen, GraphQL), WooCommerce, BigCommerce, catalog, checkout, OMS, ASO, CRO |
| `startup-analyst.md` | `agents/product/` | SaaS metrics, unit economics, pricing, GTM, fundraising, cap tables, TAM/SAM/SOM, board reporting |
| `customer-success-expert.md` | `agents/product/` | Onboarding, health scoring, churn, expansion, QBRs, NPS/CSAT, support automation, advocacy |
| `sales-automation-expert.md` | `agents/product/` | Cold email, sequencing, deliverability, CRM automation, lead scoring (BANT/MEDDIC), pipeline |
| `saas-integrations-expert.md` | `agents/product/` | OAuth, webhooks, idempotency, sync patterns, 100+ SaaS APIs (Slack, Notion, Salesforce, Stripe, etc.) |
| `workflow-automation-expert.md` | `agents/product/` | n8n, Zapier, Make, Pipedream, Activepieces — workflow design, error handling, observability, cost |
| `erp-odoo-expert.md` | `agents/product/` | Odoo modules (Python+XML), ORM, ACL, QWeb, Accounting/MRP/HR/POS, version migration, OWL, Odoo.sh |
| `fintech-payments-expert.md` | `agents/product/` | Stripe (Connect/Subscriptions/Treasury/Issuing/Tax/Radar), PayPal, Plaid, PCI, 3DS/SCA, blockchain |
| `people-operations-expert.md` | `agents/product/` | Hiring, interview kits, onboarding, perf mgmt, comp bands, equity, handbook, contracts, remote teams |
| `chief-product-officer.md` | `agents/product/` | Coordinates all 10 product agents — launch, PMF, scale, monetization playbooks |

### Security (`agents/security/`)
| Agent | Path | Key domain |
|---|---|---|
| `pentest-expert.md` | `agents/security/` | Web pentest (OWASP), Burp, recon, SQLi/XSS/SSRF/IDOR, JWT, Nuclei, Metasploit, red team, post-exploit |
| `security-architect.md` | `agents/security/` | OAuth/OIDC/SAML/passkeys, RBAC/ABAC/ReBAC, OPA, Vault, STRIDE/PASTA, CSP, WAF, SLSA/SBOM, zero trust |
| `legal-compliance-expert.md` | `agents/security/` | GDPR, CCPA, HIPAA, SOC 2, ISO 27001, PCI-DSS, FDA, EU AI Act, MSA/DPA, OSS license compliance |
| `chief-security-officer.md` | `agents/security/` | Coordinates security agents — strategy, risk mgmt, maturity model, vendor risk, IR coordination |

### Specialists (`agents/specialists/`)
| Agent | Path | Key domain |
|---|---|---|
| `game-dev-expert.md` | `agents/specialists/` | Unity, Unreal, Godot, Bevy, shaders, physics, AI, netcode, asset pipeline, profiling |
| `health-wellness-expert.md` | `agents/specialists/` | Nutrition, fitness, mental health, sleep, biomarkers, habits — educational only |
| `office-automation-expert.md` | `agents/specialists/` | docx/xlsx/pptx/pdf, OCR, LibreOffice headless, mail merge, invoice processing, batch |
| `search-expert.md` | `agents/specialists/` | Algolia, ES/OpenSearch, Typesense, Meilisearch, Exa/Tavily, relevance, autocomplete, hybrid |
| `enterprise-operations-expert.md` | `agents/specialists/` | Logistics, route optimization, trade compliance, energy/PPAs, supply chain, EDI, industrial IoT |
| `conversational-agent-expert.md` | `agents/specialists/` | Discord, Slack, Telegram, WhatsApp, Alexa, intent classification, dialogue mgmt, RAG chatbots |
| `cms-expert.md` | `agents/specialists/` | WordPress (themes/plugins/Gutenberg), WooCommerce, headless (Sanity/Strapi/Payload), Moodle |
| `reverse-engineering-expert.md` | `agents/specialists/` | Ghidra/IDA/radare2, disassembly, debugging, Frida, malware analysis — defensive/research only |

---

## ✅ Phase 6 — Department Leads + COO (COMPLETE)

All 8 master coordinator agents created:

| Agent | File | Routes to |
|---|---|---|
| `ai-cto.md` | `agents/core/` | ai-ml-expert, ai-platform-expert, orchestration-expert, data-scientist-expert |
| `software-cto.md` | `agents/core/` | All 17 software engineering agents |
| `chief-content-officer.md` | `agents/content/` | All 9 content agents |
| `chief-marketing-officer.md` | `agents/marketing/` | All 5 marketing agents |
| `chief-design-officer.md` | `agents/design/` | ui-design-expert, brand-expert, presentation-expert |
| `chief-product-officer.md` | `agents/product/` | All 10 product agents |
| `chief-security-officer.md` | `agents/security/` | pentest-expert, security-architect, legal-compliance-expert |
| `company-coo.md` | `agents/core/` | All 8 department leads |

---

## ✅ Phase 7 — New Commands, Contexts, Documentation (COMPLETE)

### New Commands (7) — DONE
| File | Command | Routes to |
|---|---|---|
| `commands/content/youtube.md` | `/youtube` | youtube-content-expert |
| `commands/content/blog.md` | `/blog` | blog-writing-expert |
| `commands/content/social.md` | `/social` | social-media-expert |
| `commands/marketing/seo.md` | `/seo` | seo-expert |
| `commands/marketing/campaign.md` | `/campaign` | chief-marketing-officer |
| `commands/design/ui.md` | `/ui` | ui-design-expert |
| `commands/core/company.md` | `/company` | company-coo (master router) |

### New Context Files (3) — DONE
| File | When loaded |
|---|---|
| `contexts/content.md` | Content/media sessions — 10 content agents listed |
| `contexts/marketing.md` | Marketing/SEO sessions — 6 marketing agents listed |
| `contexts/design.md` | Design/UI sessions — 4 design agents listed |

### Documentation Updates — DONE
- ✅ `BRAIN.md`: agent count 24 → 85, skill count 1,207 → 1,219, `_studio/` folder added, command count 51 → 58, contexts 3 → 6, full directory tree refreshed, file counts table updated, date bumped to 2026-04-10
- ✅ `CLAUDE.md`: skill count 1,191 → 1,219, `_studio/` added to categories tree + Available Content table, full Agents section rewritten (85 agents across 13 categories), Commands section expanded (58), Contexts section added, "Adding Skills" pointer updated to `skills/_studio/generate-claude-md/`

---

## Tier 2 Skill Removals (Execute after corresponding agent is built and verified)

~47 skills to remove after their agent is verified. Full list in `BRAIN.md` or the original plan file.

Key groups:
- After `ai-ml-expert`: 13 data-science-ml/ duplicates
- After `seo-expert`: 7 seo/ duplicates  
- After `ui-design-expert`: 16 ui-design/ duplicates
- After `test-expert`: 6 testing-quality/ duplicates
- After `orchestration-expert`: 2 agents-orchestration/ duplicates

---

## Agent Design Rules (for all new agents)

1. **Self-contained**: All skill knowledge embedded inline — no runtime skill file references
2. **Autonomous**: One context-gathering step upfront, then executes without interruption
3. **Deliverable-focused**: Returns complete, production-ready output
4. **Intent detection**: Agent detects task type and routes to correct internal workflow
5. **Tools**: Declare only tools actually needed
6. **MCP section**: Include `## MCP Tools Used` listing relevant servers

---

## Expected Final State

| Metric | Before | After |
|---|---|---|
| Agents | 18 | ~65 |
| New skills created | 0 | 16 |
| Skills removed (Tier 1) | 0 | 4 (done) |
| Skills to remove (Tier 2) | 0 | ~47 (pending) |
| Department coverage | 0 | 8 departments |
| Entry point routing | Manual | Automatic via generate-claude-md |
