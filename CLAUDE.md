# CLAUDE.md

This file provides guidance to Claude Code when working inside this repository.

## Project Overview

This is a **Claude Agent Skill Library** — a distributable collection of skills, agents, commands, rules, hooks, contexts, and MCP configs. It is a pure library (no build system, no test runner). Content here is installed into other projects, not used in this repo directly.

---

## Repository Structure

```
claude_kit/
├── .claude/
│   ├── package-manager.json              # Package manager config (npm)
│   ├── homunculus/instincts/inherited/   # Curated instincts for this repo
│   └── skills/claude-kit/SKILL.md    # Meta-skill for working in this repo
│
├── skills/          # 1,228 skill modules organized by category (OS/Systems Userland category folders are phased in over the agentic-OS kit build — Phase 1 shipped)
│   ├── _studio/                  # (5) ENTRY POINTS — generate-claude-md, new-features-updates, route-agents, brainstorming, batch-tasks
│   ├── core/                     # (5) continuous-learning, eval-harness, skill-stocktake, strategic-compact, configure-ecc
│   ├── development/              # (11) code-writing, build-website, api-design, systematic-debugging, ...
│   ├── planning/                 # (4) planning-specification, autonomous-agents, multi-model, ...
│   ├── testing-quality/          # (24) tdd-workflow, security-review, e2e-testing, playwright, ...
│   ├── data-backend/             # (25) postgres-patterns, database-migrations, clickhouse-io, nosql, ...
│   ├── data-science-ml/          # (60) ai-engineer, hugging-face, langchain, rag, llm-app, ...
│   ├── languages/                # (41) golang, python, kotlin, java, swift, cpp, rust, typescript, ...
│   ├── ai-platform/              # (9) claude-developer-platform, notebooklm, claude-d3js, ...
│   ├── research-docs/            # (29) research, document-writing, wiki, blog-writing, ...
│   ├── devops/                   # (20) terminal-cli-devops, git-worktrees, cicd, github-actions, ...
│   ├── devtools/                 # (18) git-hooks, jq, tmux, gdb-cli, obsidian, ...
│   ├── ui-design/                # (49) presentations, ui-ux-pro-max, design-system, hig, accessibility, ...
│   ├── integrations/             # (107) slack, hubspot, jira, zapier, notion, salesforce, ...
│   ├── integrations-scraping/    # (15) apify, firecrawl, web-scraper, ...
│   ├── frameworks-frontend/      # (49) react, angular, nextjs, svelte, tailwind, threejs, ...
│   ├── frameworks-backend/       # (24) django, fastapi, laravel, nestjs, prisma, nodejs, ...
│   ├── frameworks-mobile/        # (11) flutter, react-native, ios, expo, ...
│   ├── frameworks-desktop/       # (24) electron, avalonia, makepad, robius, progressive-web-app, ...
│   ├── cloud/                    # (24) aws, gcp, terraform, cloudflare, serverless, ...
│   ├── cloud-azure/              # (122) azure SDK skills across all Azure services
│   ├── containers-orchestration/ # (11) docker, kubernetes, helm, istio, linkerd, ...
│   ├── security-offensive/       # (36) pentesting, burpsuite, metasploit, red-team, ...
│   ├── security-defensive/       # (29) auth, secrets-management, incident-response, semgrep, ...
│   ├── agents-orchestration/     # (37) agent-framework, agent-memory, conductor, multi-advisor, ...
│   ├── architecture/             # (17) microservices, ddd, software-architecture, senior-architect, ...
│   ├── observability/            # (8) distributed-tracing, prometheus, slo-implementation, ...
│   ├── seo/                      # (31) seo-technical, seo-content, seo-audit, schema-markup, ...
│   ├── marketing-growth/         # (31) content-strategy, growth-engine, cold-email, paid-ads, ...
│   ├── product-business/         # (23) product-manager, startup, saas, analytics, ...
│   ├── ecommerce/                # (7) shopify, woocommerce, inventory, app-store, ...
│   ├── erp-odoo/                 # (24) odoo modules: accounting, hr, ecommerce, migrations, ...
│   ├── health-wellness/          # (16) nutrition, fitness, mental-health, sleep, ...
│   ├── legal-compliance/         # (15) legal-advisor, fda, gdpr, employment-contracts, ...
│   ├── fintech-payments/         # (7) stripe, paypal, plaid, payment-integration, ...
│   ├── blockchain-web3/          # (8) solidity, defi, nft, lightning-network, ...
│   ├── game-dev/                 # (9) unity, godot, unreal, bevy, shader, ...
│   ├── scientific/               # (13) astropy, biopython, matplotlib, scikit-learn, ...
│   ├── functional-programming/   # (15) fp-ts patterns: pipe, either, option, taskeither, ...
│   ├── media-video/              # (10) remotion, comfyui, podcast, audio-transcriber, ...
│   ├── office-documents/         # (7) docx, pptx, xlsx, pdf, libreoffice, ...
│   ├── bots/                     # (7) discord-bot, slack-bot, telegram-bot, alexa, ...
│   ├── personas/                 # (15) bill-gates, elon-musk, andrej-karpathy, uncle-bob, ...
│   ├── team-hr/                  # (8) hr-pro, interview-coach, team-collaboration, ...
│   ├── cms/                      # (4) wordpress, wordpress-plugin, moodle, ...
│   ├── search/                   # (5) algolia, exa-search, tavily, openapi-spec, ...
│   ├── performance/              # (4) web-performance, app-performance, ...
│   ├── baas/                     # (2) firebase, upstash, ...
│   ├── domain/                   # (5) logistics, trade-compliance, energy-procurement, ...
│   ├── os-linux-platform/        # (5) linux-kernel-config-for-custom-os, hardware-profiling-and-driver-mapping, drm-kms-and-mesa-basics, pipewire-wireplumber-session, input-stack-libinput-udev — Phase 1 shipped; Phase 8 adds arch-portability-and-hal-boundary
│   ├── os-boot-and-firmware/     # (0) [scaffolded — Phase 2] UEFI/systemd-boot, secure boot, TPM2, initramfs
│   ├── os-system-services/       # (0) [scaffolded — Phase 4] PID 1, cgroups v2, seccomp+landlock, systemd user units
│   ├── os-distribution/          # (0) [scaffolded — Phase 5] image-based atomic distro, package signing, A/B rollback
│   ├── os-desktop-stack/         # (0) [scaffolded — Phase 3] Smithay compositor, gestures, portals, HDR, fractional scaling
│   └── specialized/              # (112) niche/branded skills, reverse-engineering, forensics, ...
├── agents/          # 83 agents across board + 3 operating companies — holding company OS
│   ├── board/                          # (4) corporate governance — sits above the operating companies
│   │   ├── company-coo.md              # master router across all 3 operating companies
│   │   ├── chief-of-staff.md           # cross-company ops & comms triage
│   │   ├── chief-design-officer.md     # cross-company design coherence (UI ↔ brand ↔ visuals)
│   │   └── people-operations-expert.md # corporate HR — hiring, comp, contracts, handbook
│   │
│   ├── software-company/               # (61) builds & ships software — CEO: software-cto
│   │   ├── software-cto.md             # CEO of software-company
│   │   ├── engineering/                # (17) planner, architect, software-developer, web-front/back, mobile, desktop, mcp-server, systems, python, typescript, polyglot, cinematic-website-builder, code-reviewer, refactor-cleaner, doc-updater, build-error-resolver
│   │   ├── ai/                         # (5) ai-cto + ai-ml, ai-platform, orchestration, data-scientist
│   │   ├── devops/                     # (4) devops-infra, cloud-architect, azure, observability-engineer
│   │   ├── data/                       # (2) database-architect, database-reviewer
│   │   ├── qa/                         # (4) test-expert, tdd-guide, e2e-runner, security-reviewer
│   │   ├── languages/                  # (5) go-reviewer, go-build-resolver, kotlin-reviewer, kotlin-build-resolver, python-reviewer
│   │   ├── product/                    # (10) chief-product-officer + product-mgr, ecommerce, startup-analyst, customer-success, sales-automation, saas-integrations, workflow-automation, erp-odoo, fintech-payments
│   │   ├── design/                     # (1) ui-design-expert
│   │   ├── security/                   # (4) chief-security-officer + pentest, security-architect, legal-compliance
│   │   ├── specialists/                # (7) game-dev, office-automation, search, enterprise-operations, conversational-agent, cms, reverse-engineering
│   │   └── os-engineering/             # (2) os-userland-architect (sub-lead) + linux-platform-expert (Phase 1 shipped); division continues growing in kit phases 3,5,8
│   │
│   ├── marketing-company/              # (7) marketing services — CEO: chief-marketing-officer
│   │   ├── chief-marketing-officer.md
│   │   ├── strategy/                   # (3) seo, growth, competitor-intelligence
│   │   ├── campaigns/                  # (2) paid-ads, email-marketing
│   │   └── brand/                      # (1) brand-expert
│   │
│   └── media-company/                  # (11) content & media production — CEO: chief-content-officer
│       ├── chief-content-officer.md
│       ├── video/                      # (2) youtube, video-production
│       ├── audio/                      # (1) podcast
│       ├── editorial/                  # (3) blog, newsletter, technical-writer
│       ├── visual/                     # (2) image-creation, presentation
│       └── distribution/               # (2) social-media, content-repurposing
├── commands/        # 58 slash command definitions across 9 categories
│   ├── core/            # learn, skill-create, sessions, task-handoff, wrapup, instinct-*, eval, harness-audit, company, ...
│   ├── development/     # code-review, build-fix, refactor-clean, verify, update-*, pm2, ...
│   ├── testing-quality/ # tdd, e2e, quality-gate, test-coverage
│   ├── planning/        # plan, orchestrate, model-route, loop-*, multi-*, brainstorm
│   ├── languages/       # go-*, kotlin-*, gradle-build, python-review
│   ├── content/         # youtube, blog, social
│   ├── marketing/       # seo, campaign
│   ├── design/          # ui
│   └── specialized/     # claw
├── rules/           # common/ + 8 language dirs (golang, python, kotlin, ...)
├── contexts/        # dev.md, research.md, review.md, content.md, marketing.md, design.md
├── hooks/           # hooks.json (PreToolUse, PostToolUse, SessionStart, ...)
├── mcp-configs/     # mcp-servers.json (14 MCP integrations, tokens sanitized)
│
├── everything-claude-code/   # Source repo cloned for reference (do not edit)
└── CLAUDE.md
```

**Design principle:** `.claude/` is minimal — repo meta only. All distributable content lives at root level, mirroring the ECC library architecture.

---

## How to Use This Library

### Zero-Copy Reference Architecture

Projects **never copy files** from this library. Instead, they reference it directly:

- **Skills** → absolute `@` imports in the project's `.claude/CLAUDE.md` (only selected skills load into context)
- **Agents, commands, hooks, contexts, rules** → Windows directory junctions (`mklink /J`) pointing to this kit's directories (zero disk usage, always up to date)

**Result:** Edit any file in the kit → all projects using it see the change automatically on the next session.

### In a new project — generate a full Claude Code setup

From any project directory, ask Claude:

```
/generate-claude-md

I'm building a [description of your project].
```

The `generate-claude-md` skill (now in `skills/_studio/`) will:
1. Analyze your project description
2. Select 2–6 relevant skills for the project type
3. Create in your project:
   - `.claude/CLAUDE.md` — with `@` imports for selected skills (absolute kit paths)
   - `.claude/CLAUDE.planning.md` — planning phase guide
   - `.spec/` — planning artifacts
   - `.claude/project-config.md` — kit path + skill list
   - Directory junctions: `.claude/agents/` → kit, `.claude/commands/` → kit, `.claude/rules/common/` → kit, `.claude/hooks/` → kit, `.claude/contexts/` → kit

No files are copied. The kit path is recorded in `project-config.md` for future updates.

### Adding a skill to an existing project

Edit `.claude/CLAUDE.md` and add one line:
```
@C:/Users/Hp/Desktop/Experiment/claude_kit/skills/<category>/<skill-name>/SKILL.md
```
Update `.claude/project-config.md` to record the addition. Active on next session start.

### Load curated instincts for this repo

When working inside the Skill Builder repository itself:

```
/instinct-import .claude/homunculus/instincts/inherited/skill-builder-instincts.yaml
```

---

## Available Content

### Skills (1,224 across 52 categories)

**Studio / Entry Points** (5)
`generate-claude-md` · `new-features-updates` · `route-agents` · `brainstorming` · `batch-tasks` — these route project requests to the right department agents and ideation flows

**Core / Meta** (6)
`continuous-learning` · `strategic-compact` · `eval-harness` · `skill-stocktake` · `configure-ecc` · `karpathy-principles`

**Development** (11)
`code-writing-software-development` · `build-website-web-app` · `api-design` · `systematic-debugging` · `branch-completion` · `api-endpoint-builder` · `framework-migration-*` · ...

**Planning & Architecture** (4 + 17 architecture)
`planning-specification-architecture-software` · `planning-specification-architecture-marketing` · `planning-specification-architecture-media` · `autonomous-agents-task-automation` · `parallel-agent-dispatch` · `multi-model-model-agnostic-platforms`
Architecture: `microservices-patterns` · `ddd-*` · `software-architecture` · `senior-architect` · `backend-architect` · ...

**Testing & Quality** (24)
`tdd-workflow` · `security-review` · `e2e-testing` · `playwright-*` · `k6-load-testing` · `ab-test-setup` · `acceptance-orchestrator` · `lint-and-validate` · ...

**Data & Backend** (25)
`postgres-patterns` · `database-migrations` · `clickhouse-io` · `database-admin` · `nosql-expert` · `neon-postgres` · `vector-database-engineer` · `cqrs-implementation` · `event-sourcing-architect` · ...

**Data Science & ML** (60)
`ai-engineer` · `ml-engineer` · `rag-*` · `langchain-*` · `langgraph` · `hugging-face-*` · `llm-app-patterns` · `prompt-engineering` · `computer-vision-expert` · `crewai` · `voice-ai-*` · ...

**Languages** (41)
`golang-patterns` · `python-patterns` · `kotlin-patterns` · `java-patterns` · `swift-patterns` · `cpp-patterns` · `rust-pro` · `typescript-*` · `javascript-*` · `csharp-pro` · `scala-pro` · `haskell-pro` · `dotnet-*` · ...

**Frameworks — Frontend** (49)
`react-*` · `angular-*` · `nextjs-*` · `sveltekit` · `tailwind-*` · `shadcn` · `threejs-*` · `animejs-*` · `tanstack-query` · `zustand-*` · ...

**Frameworks — Backend** (24)
`django-pro` · `fastapi-*` · `laravel-expert` · `nestjs-expert` · `prisma-expert` · `drizzle-orm` · `nodejs-*` · `bullmq-*` · `inngest` · ...

**Frameworks — Mobile** (11)
`flutter-expert` · `react-native-*` · `ios-*` · `expo-*` · `mobile-developer` · ...

**Frameworks — Desktop** (24)
`electron-development` · `avalonia-*` · `makepad-*` · `robius-*` · `progressive-web-app` · ...

**Claude / AI Platform** (9)
`claude-developer-platform` · `notebooklm` · `claude-d3js-skill` · `claude-in-chrome-troubleshooting` · ...

**Agents & Orchestration** (37)
`agent-framework-*` · `agent-memory-*` · `agent-orchestrator` · `conductor-*` · `computer-use-agents` · `multi-advisor` · `subagent-driven-development` · ...

**Cloud** (24)
`aws-serverless` · `aws-cost-optimizer` · `gcp-cloud-run` · `terraform-*` · `cloudflare-workers` · `cloudformation-*` · ...

**Cloud — Azure** (122)
Azure SDK skills across all services: AI, identity, storage, cosmos, monitor, keyvault, eventhub, servicebus, search, ...

**Containers & Orchestration** (11)
`docker-expert` · `kubernetes-*` · `helm-chart-scaffolding` · `istio-*` · `linkerd-*` · `service-mesh-*` · ...

**Security — Offensive** (36)
`pentest-*` · `burpsuite-*` · `metasploit-*` · `red-team-*` · `sql-injection-*` · `xss-*` · `vulnerability-scanner` · ...

**Security — Defensive** (29)
`auth-implementation-patterns` · `secrets-management` · `incident-response-*` · `semgrep-*` · `privacy-by-design` · `threat-modeling-*` · ...

**SEO** (32)
`seo-technical` · `seo-content-*` · `seo-audit` · `seo-keyword-strategist` · `schema-markup` · `programmatic-seo` · `seo-strategy` · ...

**Marketing & Growth** (31)
`content-strategy` · `growth-engine` · `cold-email` · `paid-ads` · `lead-magnets` · `viral-generator-builder` · ...

**Product & Business** (23)
`product-manager` · `startup-*` · `saas-*` · `analytics-*` · `kpi-dashboard-design` · `pricing-strategy` · ...

**Integrations** (107)
`slack-automation` · `hubspot-*` · `jira-automation` · `notion-automation` · `zapier-*` · `n8n-*` · `google-*` · `salesforce-automation` · `stripe-automation` · ... (107 SaaS integrations)

**Integrations — Scraping** (15)
`apify-*` · `firecrawl-scraper` · `web-scraper` · ...

**DevOps** (20)
`terminal-cli-devops` · `git-worktrees` · `github-actions-*` · `gitlab-ci-*` · `gitops-workflow` · `monorepo-*` · ...

**DevTools** (18)
`git-hooks-automation` · `jq` · `tmux` · `gdb-cli` · `obsidian-*` · `uv-package-manager` · ...

**Research & Docs** (29)
`research-information-retreival` · `document-content-writing-editing` · `wiki-*` · `blog-writing-guide` · `scientific-writing` · `changelog-automation` · ...

**UI & Design** (52)
`presentations-ui-design` · `ui-ux-pro-max` · `design-system` · `hig-*` · `accessibility-*` · `landing-page-generator` · `cinematic-website-builder` · `website-intelligence` · `asset-generation` · `3d-animation-creator` · `spline-3d-integration` · ...

**Observability** (8)
`distributed-tracing` · `prometheus-configuration` · `slo-implementation` · `observability-engineer` · ...

**ERP — Odoo** (24)
`odoo-module-developer` · `odoo-orm-expert` · `odoo-migration-helper` · `odoo-ecommerce-configurator` · ... (24 Odoo modules)

**Scientific** (13)
`astropy` · `biopython` · `matplotlib` · `scikit-learn` · `plotly` · `polars` · `qiskit` · `networkx` · ...

**Functional Programming** (15)
`fp-ts-*` patterns: pipe, either, option, taskeither, backend, react, refactor, ...

**E-commerce** (7)
`shopify-*` · `wordpress-woocommerce` · `inventory-demand-planning` · `app-store-optimization` · ...

**Health & Wellness** (16)
`nutrition-analyzer` · `fitness-analyzer` · `mental-health-analyzer` · `sleep-analyzer` · `tcm-constitution-analyzer` · ...

**Legal & Compliance** (15)
`legal-advisor` · `fda-*` · `employment-contract-templates` · `gdpr-data-handling` · ...

**Fintech & Payments** (7)
`stripe-integration` · `paypal-integration` · `plaid-fintech` · `payment-integration` · ...

**Blockchain & Web3** (8)
`blockchain-developer` · `defi-protocol-templates` · `nft-standards` · `solidity-security` · ...

**Game Development** (9)
`unity-*` · `godot-*` · `unreal-engine-cpp-pro` · `bevy-ecs-expert` · `shader-programming-glsl` · ...

**Media & Video** (10)
`remotion-*` · `comfyui-gateway` · `podcast-generation` · `audio-transcriber` · `stability-ai` · ...

**Office & Documents** (7)
`docx-official` · `pptx-official` · `xlsx-official` · `pdf-official` · `libreoffice` · ...

**Bots** (7)
`discord-bot-architect` · `slack-bot-builder` · `telegram-bot-builder` · `amazon-alexa` · `chat-widget` · ...

**Personas** (15)
`bill-gates` · `elon-musk` · `steve-jobs` · `andrej-karpathy` · `geoffrey-hinton` · `yann-lecun` · `uncle-bob-craft` · ...

**Team & HR** (8)
`hr-pro` · `interview-coach` · `team-collaboration-*` · `daily-news-report` · ...

**Performance** (4) · **CMS** (4) · **Search** (5) · **BaaS** (2) · **Customer Support** (1) · **Salesforce** (1) · **Domain** (5)

**OS / Systems Userland** (5, Phase 1 shipped — more in Phases 2–8)
`linux-kernel-config-for-custom-os` · `hardware-profiling-and-driver-mapping` · `drm-kms-and-mesa-basics` · `pipewire-wireplumber-session` · `input-stack-libinput-udev` — the L1-wrapping layer for a custom agentic OS. Used with `os-userland-architect` + `linux-platform-expert` agents.

**Specialized** (112)
Niche/branded skills, reverse-engineering, forensics, custom workflows, and community contributions

### Agents (84 across board + 3 operating companies)

The agent suite is structured as a holding company. The **board** contains cross-company governance roles. Underneath, three **operating companies** each have their own CEO and internal cascade. Route through `company-coo` (master) for multi-company work, or directly to a company CEO for single-company work. Each CEO manages their own internal org — do not reach past a CEO into their specialists.

#### Board (4) — `agents/board/`
`company-coo` (master router across all 3 companies) · `chief-of-staff` (cross-company ops + comms triage) · `chief-design-officer` (cross-company design coherence) · `people-operations-expert` (corporate HR)

#### software-company (62) — `agents/software-company/` — CEO: `software-cto`

**Engineering** (17, `engineering/`) — `architect` · `planner` · `software-developer-expert` · `web-frontend-expert` · `web-backend-expert` · `mobile-expert` · `desktop-expert` · `systems-programming-expert` · `mcp-server-expert` · `python-expert` · `typescript-expert` · `polyglot-expert` · `cinematic-website-builder` · `code-reviewer` · `refactor-cleaner` · `doc-updater` · `build-error-resolver`

**AI / ML** (5, `ai/`) — sub-lead: `ai-cto` · `ai-ml-expert` · `ai-platform-expert` · `orchestration-expert` · `data-scientist-expert`

**DevOps** (4, `devops/`) — `devops-infra-expert` · `cloud-architect` · `azure-expert` · `observability-engineer`

**Data** (2, `data/`) — `database-architect` · `database-reviewer`

**QA** (4, `qa/`) — `test-expert` · `tdd-guide` · `e2e-runner` · `security-reviewer`

**Languages** (5, `languages/`) — `go-reviewer` · `go-build-resolver` · `kotlin-reviewer` · `kotlin-build-resolver` · `python-reviewer`

**Product & Business** (10, `product/`) — sub-lead: `chief-product-officer` · `product-manager-expert` · `ecommerce-expert` · `startup-analyst` · `customer-success-expert` · `sales-automation-expert` · `saas-integrations-expert` · `workflow-automation-expert` · `erp-odoo-expert` · `fintech-payments-expert`

**Software UI Design** (1, `design/`) — `ui-design-expert`

**Security & Compliance** (4, `security/`) — sub-lead: `chief-security-officer` · `pentest-expert` · `security-architect` · `legal-compliance-expert`

**Specialists** (7, `specialists/`) — `game-dev-expert` · `office-automation-expert` · `search-expert` · `enterprise-operations-expert` · `conversational-agent-expert` · `cms-expert` · `reverse-engineering-expert`

**OS Engineering** (2, `os-engineering/`) — sub-lead: `os-userland-architect` (master agent for custom-OS / userland / desktop runtime / agentic-OS work; owns L2 system services, L3 desktop runtime, L4 agentic layer, capability model, HAL boundary, x86_64 ↔ aarch64 portability) · `linux-platform-expert` (Phase 1 — L1 wrapping: kernel config, hardware profiling, DRM/KMS, libinput, PipeWire, NetworkManager, udev, firmware). Division continues growing in kit phases 3, 5, 8 with `wayland-compositor-expert`, `immutable-distro-expert`, `asahi-porting-expert`.

#### marketing-company (7) — `agents/marketing-company/` — CEO: `chief-marketing-officer`

**Strategy** (3, `strategy/`) — `seo-expert` · `growth-marketing-expert` · `competitor-intelligence-expert`

**Campaigns** (2, `campaigns/`) — `paid-ads-expert` · `email-marketing-expert`

**Brand** (1, `brand/`) — `brand-expert`

#### media-company (11) — `agents/media-company/` — CEO: `chief-content-officer`

**Video** (2, `video/`) — `youtube-content-expert` · `video-production-expert`

**Audio** (1, `audio/`) — `podcast-expert`

**Editorial** (3, `editorial/`) — `blog-writing-expert` · `newsletter-expert` · `technical-writer-expert`

**Visual** (2, `visual/`) — `image-creation-expert` · `presentation-expert`

**Distribution** (2, `distribution/`) — `social-media-expert` · `content-repurposing-expert`

### Commands (58)

**Core** — `learn` · `learn-eval` · `skill-create` · `evolve` · `eval` · `checkpoint` · `save-session` · `resume-session` · `task-handoff` · `wrapup` · `sessions` · `aside` · `instinct-export` · `instinct-import` · `instinct-status` · `harness-audit` · `projects` · `promote` · `setup-pm` · `company`

**Development** — `code-review` · `build-fix` · `refactor-clean` · `verify` · `update-docs` · `update-codemaps` · `pm2` · `prompt-optimize`

**Testing & Quality** — `tdd` · `e2e` · `quality-gate` · `test-coverage`

**Planning** — `plan` · `orchestrate` · `model-route` · `loop-start` · `loop-status` · `multi-plan` · `multi-execute` · `multi-workflow` · `multi-backend` · `multi-frontend` · `brainstorm`

**Languages** — `go-build` · `go-review` · `go-test` · `gradle-build` · `kotlin-build` · `kotlin-review` · `kotlin-test` · `python-review`

**Content** — `youtube` · `blog` · `social`

**Marketing** — `seo` · `campaign`

**Design** — `ui`

**Specialized** — `claw`

### Contexts (6)
`dev.md` · `review.md` · `research.md` · `content.md` · `marketing.md` · `design.md`

### Rules
`rules/common/` — 10 files (agents, coding-style, development-workflow, git-workflow, hooks, patterns, performance, security, testing, **token-cost**)
`rules/<lang>/` — 5 files each for: golang, kotlin, python, swift, perl, php, typescript

---

## Adding or Updating Skills

1. Create `skills/<category>/<skill-name>/SKILL.md` using the WAT format (see `.claude/skills/claude-kit/SKILL.md`)
2. Add the skill to `skills/_studio/generate-claude-md/SKILL.md` under the correct category
3. Update the Available Content table in this file
4. Check for overlap with existing skills first — merge if >50% overlap

**WAT format:** YAML frontmatter (`name` + `description` only), numbered sections, `---` separators, second-person imperative, bold `**Rule:**` callouts. Token-minimal — no preamble, no redundancy.
