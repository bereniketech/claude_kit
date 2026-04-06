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
├── skills/          # 1,191 skill modules organized by category
│   ├── core/                     # (7) generate-claude-md, continuous-learning, eval-harness, ...
│   ├── development/              # (11) code-writing, build-website, api-design, systematic-debugging, ...
│   ├── planning/                 # (5) planning-specification, autonomous-agents, multi-model, ...
│   ├── testing-quality/          # (24) tdd-workflow, security-review, e2e-testing, playwright, ...
│   ├── data-backend/             # (25) postgres-patterns, database-migrations, clickhouse-io, nosql, ...
│   ├── data-science-ml/          # (60) ai-engineer, hugging-face, langchain, rag, llm-app, ...
│   ├── languages/                # (41) golang, python, kotlin, java, swift, cpp, rust, typescript, ...
│   ├── ai-platform/              # (9) claude-developer-platform, notebooklm, claude-d3js, ...
│   ├── research-docs/            # (29) research, document-writing, wiki, blog-writing, ...
│   ├── devops/                   # (20) terminal-cli-devops, git-worktrees, cicd, github-actions, ...
│   ├── devtools/                 # (18) git-hooks, jq, tmux, gdb-cli, obsidian, ...
│   ├── ui-design/                # (48) presentations, ui-ux-pro-max, design-system, hig, accessibility, ...
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
│   └── specialized/              # (112) niche/branded skills, reverse-engineering, forensics, ...
├── agents/          # 18 agent definitions, organized by category
│   ├── core/            # planner, architect, chief-of-staff, loop-operator, harness-optimizer
│   ├── development/     # code-reviewer, refactor-cleaner, doc-updater, build-error-resolver
│   ├── testing-quality/ # tdd-guide, security-reviewer, e2e-runner
│   ├── data-backend/    # database-reviewer
│   └── languages/       # go-reviewer, go-build-resolver, kotlin-reviewer, kotlin-build-resolver, python-reviewer
├── commands/        # 50 slash command definitions, organized by category
│   ├── core/            # learn, skill-create, sessions, task-handoff, wrapup, instinct-*, eval, harness-audit, ...
│   ├── development/     # code-review, build-fix, refactor-clean, verify, update-*, pm2, ...
│   ├── testing-quality/ # tdd, e2e, quality-gate, test-coverage
│   ├── planning/        # plan, orchestrate, model-route, loop-*, multi-*, brainstorm
│   ├── languages/       # go-*, kotlin-*, gradle-build, python-review
│   └── specialized/     # claw
├── rules/           # common/ + 8 language dirs (golang, python, kotlin, ...)
├── contexts/        # dev.md, research.md, review.md
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

The `generate-claude-md` skill will:
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

### Skills (1,191 across 49 categories)

**Core / Meta** (7)
`generate-claude-md` · `new-features-updates` · `skill-stocktake` · `configure-ecc` · `continuous-learning` · `strategic-compact` · `eval-harness`

**Development** (11)
`code-writing-software-development` · `build-website-web-app` · `api-design` · `systematic-debugging` · `branch-completion` · `api-endpoint-builder` · `framework-migration-*` · ...

**Planning & Architecture** (5 + 17 architecture)
`planning-specification-architecture` · `autonomous-agents-task-automation` · `brainstorming` · `parallel-agent-dispatch` · `multi-model-model-agnostic-platforms`
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

**SEO** (31)
`seo-technical` · `seo-content-*` · `seo-audit` · `seo-keyword-strategist` · `schema-markup` · `programmatic-seo` · ...

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

**UI & Design** (48)
`presentations-ui-design` · `ui-ux-pro-max` · `design-system` · `hig-*` · `accessibility-*` · `landing-page-generator` · `spline-3d-integration` · ...

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

**Specialized** (112)
Niche/branded skills, reverse-engineering, forensics, custom workflows, and community contributions

### Agents (18)

**Core** — `planner` · `architect` · `chief-of-staff` · `loop-operator` · `harness-optimizer`

**Development** — `code-reviewer` · `refactor-cleaner` · `doc-updater` · `build-error-resolver`

**Testing & Quality** — `tdd-guide` · `security-reviewer` · `e2e-runner`

**Data & Backend** — `database-reviewer`

**Languages** — `go-reviewer` · `go-build-resolver` · `kotlin-reviewer` · `kotlin-build-resolver` · `python-reviewer`

### Commands (51)

**Core** — `learn` · `learn-eval` · `skill-create` · `evolve` · `eval` · `checkpoint` · `save-session` · `resume-session` · `task-handoff` · `wrapup` · `sessions` · `aside` · `instinct-export` · `instinct-import` · `instinct-status` · `harness-audit` · `projects` · `promote` · `setup-pm`

**Development** — `code-review` · `build-fix` · `refactor-clean` · `verify` · `update-docs` · `update-codemaps` · `pm2` · `prompt-optimize`

**Testing & Quality** — `tdd` · `e2e` · `quality-gate` · `test-coverage`

**Planning** — `plan` · `orchestrate` · `model-route` · `loop-start` · `loop-status` · `multi-plan` · `multi-execute` · `multi-workflow` · `multi-backend` · `multi-frontend` · `brainstorm`

**Languages** — `go-build` · `go-review` · `go-test` · `gradle-build` · `kotlin-build` · `kotlin-review` · `kotlin-test` · `python-review`

**Specialized** — `claw`

### Rules
`rules/common/` — 9 files (agents, coding-style, development-workflow, git-workflow, hooks, patterns, performance, security, testing)
`rules/<lang>/` — 5 files each for: golang, kotlin, python, swift, perl, php, typescript

---

## Adding or Updating Skills

1. Create `skills/<category>/<skill-name>/SKILL.md` using the WAT format (see `.claude/skills/claude-kit/SKILL.md`)
2. Add the skill to `skills/core/generate-claude-md/SKILL.md` under the correct category
3. Update the Available Content table in this file
4. Check for overlap with existing skills first — merge if >50% overlap

**WAT format:** YAML frontmatter (`name` + `description` only), numbered sections, `---` separators, second-person imperative, bold `**Rule:**` callouts. Token-minimal — no preamble, no redundancy.
