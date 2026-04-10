# claude_kit BRAIN

> **Purpose:** Single-read knowledge base for the entire claude_kit repository. Read this instead of scanning the codebase. Updated: 2026-04-10.

---

## 1. What This Repo Is

A **distributable Claude Code skill library** — a collection of skills, agents, commands, rules, hooks, contexts, and MCP configs. Pure library: no build system, no test runner. Content is installed into other projects via zero-copy references (Windows directory junctions + absolute `@` imports).

**Owner:** bereniketech | **Path:** `C:/Users/Hp/Desktop/Experiment/claude_kit`

---

## 2. Architecture — Zero-Copy Reference Model

Projects **never copy files** from this kit. Instead:

| Component | Distribution Method | Cost |
|-----------|-------------------|------|
| **Skills** | `@` absolute imports in project's `.claude/CLAUDE.md` (only 2-6 selected per project) | Loaded into context on session start |
| **Agents** | Windows junction (`mklink /J`) from project `.claude/agents/` → kit `agents/` | Zero context cost — on-demand only |
| **Commands** | Junction from project `.claude/commands/` → kit `commands/` | Zero context cost — invoked via `/command` |
| **Rules** | Junction from project `.claude/rules/common/` → kit `rules/common/` + language-specific | Always loaded |
| **Hooks** | Junction from project `.claude/hooks/` → kit `hooks/` | Fires on events |
| **Contexts** | Junction from project `.claude/contexts/` → kit `contexts/` | On-demand |

**Effect:** Edit any file in the kit → all linked projects see changes immediately (junctions) or on next session start (`@` imports).

---

## 3. Directory Structure (Current State)

```
claude_kit/
├── .claude/
│   ├── package-manager.json              # npm preference
│   ├── homunculus/instincts/inherited/   # Curated instincts YAML
│   └── skills/claude-kit/SKILL.md       # Meta-skill (repo-editing guidance)
│
├── skills/                    # 1,219 SKILL.md files across 52 categories
│   ├── _studio/               # (2) ENTRY-POINT skills (route to agents): generate-claude-md, new-features-updates
│   ├── core/                  # (5) continuous-learning, eval-harness, skill-stocktake,
│   │                          #     strategic-compact, configure-ecc
│   ├── development/           # (11) code-writing, build-website, api-design, systematic-debugging,
│   │                          #      branch-completion, api-endpoint-builder, framework-migration, ...
│   ├── planning/              # (5) planning-specification, autonomous-agents, multi-model,
│   │                          #     brainstorming, parallel-agent-dispatch
│   ├── testing-quality/       # (24) tdd-workflow, security-review, e2e-testing, playwright,
│   │                          #      k6-load-testing, ab-test-setup, acceptance-orchestrator, bats-testing, ...
│   ├── data-backend/          # (25) postgres-patterns, database-migrations, clickhouse-io, database-admin,
│   │                          #      nosql-expert, neon-postgres, vector-database, cqrs, event-sourcing, ...
│   ├── data-science-ml/       # (60) ai-engineer, ml-engineer, rag-*, langchain-*, langgraph,
│   │                          #      hugging-face-*, llm-app-patterns, prompt-engineering, crewai, ...
│   ├── languages/             # (41) golang, python, kotlin, java, swift, cpp, rust-pro, typescript,
│   │                          #      javascript, csharp-pro, scala-pro, haskell-pro, dotnet, android, ...
│   ├── ai-platform/           # (9) claude-developer-platform, notebooklm, claude-d3js,
│   │                          #     claude-in-chrome-troubleshooting, ...
│   ├── research-docs/         # (29) research, document-writing, wiki-*, blog-writing, scientific-writing,
│   │                          #      changelog-automation, citation-management, ...
│   ├── devops/                # (20) terminal-cli-devops, git-worktrees, github-actions, gitlab-ci,
│   │                          #      gitops-workflow, monorepo, cicd-automation, bazel, circleci, ...
│   ├── devtools/              # (18) git-hooks-automation, jq, tmux, gdb-cli, obsidian,
│   │                          #      uv-package-manager, busybox-on-windows, file-organizer, ...
│   ├── ui-design/             # (49) presentations, ui-ux-pro-max, design-system, hig, accessibility,
│   │                          #      landing-page-generator, cinematic-website-builder, 3d-web-experience, canvas-design, ...
│   ├── integrations/          # (107) slack, hubspot, jira, zapier, notion, salesforce, airtable,
│   │                          #       amplitude, asana, bamboohr, activecampaign, ... (107 SaaS integrations)
│   ├── integrations-scraping/ # (15) apify-*, firecrawl-scraper, web-scraper, ...
│   ├── frameworks-frontend/   # (49) react-*, angular-*, nextjs-*, sveltekit, tailwind-*, shadcn,
│   │                          #      threejs-*, tanstack-query, zustand, ...
│   ├── frameworks-backend/    # (24) django-pro, fastapi-*, laravel, nestjs, prisma, drizzle-orm,
│   │                          #      nodejs-*, bullmq, bun-development, convex, clerk-auth, ...
│   ├── frameworks-mobile/     # (11) flutter-expert, react-native-*, ios-*, expo-*, mobile-developer, ...
│   ├── frameworks-desktop/    # (24) electron, avalonia-*, makepad-*, robius-*, progressive-web-app, ...
│   ├── agents-orchestration/  # (37) agent-framework-*, agent-memory-*, agent-orchestrator,
│   │                          #      conductor-*, multi-advisor, subagent-driven-development, ...
│   ├── architecture/          # (18) microservices-patterns, ddd-*, software-architecture,
│   │                          #      senior-architect, backend-architect, architecture-decision-records, ...
│   ├── cloud/                 # (24) aws-serverless, aws-cost-optimizer, gcp-cloud-run, terraform-*,
│   │                          #      cloudflare-workers, cloudformation-*, appdeploy, ...
│   ├── cloud-azure/           # (122) Azure SDK skills across all services: AI, identity, storage,
│   │                          #       cosmos, monitor, keyvault, eventhub, servicebus, search, ...
│   ├── containers-orchestration/ # (11) docker-expert, kubernetes-*, helm-chart, istio-*,
│   │                          #        linkerd-*, devcontainer-setup, ...
│   ├── security-offensive/    # (36) pentesting, burpsuite, metasploit, red-team, sql-injection,
│   │                          #      xss, api-fuzzing, active-directory-attacks, ...
│   ├── security-defensive/    # (29) auth-implementation-patterns, secrets-management, incident-response,
│   │                          #      semgrep, privacy-by-design, threat-modeling, ...
│   ├── seo/                   # (31) seo-technical, seo-content-*, seo-audit, schema-markup,
│   │                          #      programmatic-seo, local-legal-seo-audit, ...
│   ├── marketing-growth/      # (31) content-strategy, growth-engine, cold-email, paid-ads,
│   │                          #      lead-magnets, churn-prevention, competitor-alternatives, ...
│   ├── product-business/      # (23) product-manager, startup-*, saas-*, analytics-*,
│   │                          #      kpi-dashboard-design, billing-automation, ai-product, ...
│   ├── ecommerce/             # (7) shopify-*, woocommerce, inventory, app-store-optimization, ...
│   ├── erp-odoo/              # (24) odoo-module-developer, odoo-orm-expert, odoo-migration-helper,
│   │                          #      odoo-ecommerce, odoo-accounting, odoo-docker, odoo-tests, ...
│   ├── health-wellness/       # (16) nutrition-analyzer, fitness-analyzer, mental-health-analyzer,
│   │                          #      sleep-analyzer, tcm-constitution-analyzer, family-health, ...
│   ├── legal-compliance/      # (15) legal-advisor, fda-*, employment-contracts, gdpr, ...
│   ├── fintech-payments/      # (7) stripe-integration, paypal, plaid-fintech, payment-integration, ...
│   ├── blockchain-web3/       # (8) blockchain-developer, defi-protocol, nft-standards,
│   │                          #     solidity-security, lightning-architecture, ...
│   ├── game-dev/              # (19) unity-*, godot-*, unreal-engine, bevy-ecs, shader-programming,
│   │                          #      minecraft-bukkit, game-development, ...
│   ├── media-video/           # (10) remotion-*, comfyui-gateway, podcast-generation,
│   │                          #      audio-transcriber, stability-ai, magic-animator, ...
│   ├── office-documents/      # (11) docx-official, pptx-official, xlsx-official, pdf-official,
│   │                          #      libreoffice, json-canvas, office-productivity, ...
│   ├── bots/                  # (7) discord-bot, slack-bot, telegram-bot, alexa, chat-widget, ...
│   ├── personas/              # (15) bill-gates, elon-musk, andrej-karpathy, geoffrey-hinton,
│   │                          #      yann-lecun, uncle-bob, explain-like-socrates, ...
│   ├── team-hr/               # (8) hr-pro, interview-coach, team-collaboration, daily-news-report, ...
│   ├── scientific/            # (13) astropy, biopython, matplotlib, scikit-learn, plotly,
│   │                          #      polars, qiskit, networkx, cirq, ...
│   ├── functional-programming/ # (15) fp-ts patterns: pipe, either, option, taskeither,
│   │                          #       backend, react, refactor, data-transforms, ...
│   ├── observability/         # (8) distributed-tracing, prometheus, slo-implementation,
│   │                          #     observability-engineer, distributed-debugging, ...
│   ├── search/                # (5) algolia, exa-search, tavily, openapi-spec, search-specialist
│   ├── performance/           # (4) web-performance, app-performance, performance-engineer, ...
│   ├── cms/                   # (4) wordpress, wordpress-plugin, wordpress-theme, moodle, ...
│   ├── domain/                # (5) logistics, trade-compliance, energy-procurement,
│   │                          #     carrier-relationship, quality-nonconformance
│   ├── baas/                  # (2) firebase, upstash
│   ├── salesforce/            # (1) salesforce-development
│   ├── customer-support/      # (1) customer-support
│   └── specialized/          # (112) niche/branded skills, reverse-engineering, forensics, ...
│
├── agents/                    # 85 agent files across 13 categories
│   ├── core/                  # (8) planner, architect, chief-of-staff, loop-operator, harness-optimizer,
│   │                          #     ai-cto, software-cto, company-coo
│   ├── ai/                    # (4) ai-ml-expert, ai-platform-expert, orchestration-expert, data-scientist-expert
│   ├── content/               # (10) chief-content-officer, youtube/video/image/blog/social/podcast/
│   │                          #      newsletter/content-repurposing/technical-writer experts
│   ├── marketing/             # (6) chief-marketing-officer, seo, growth, paid-ads, email,
│   │                          #     competitor-intelligence experts
│   ├── design/                # (4) chief-design-officer, ui-design, brand, presentation experts
│   ├── product/               # (11) chief-product-officer, product-manager, ecommerce, startup-analyst,
│   │                          #      customer-success, sales-automation, saas-integrations,
│   │                          #      workflow-automation, erp-odoo, fintech-payments, people-operations
│   ├── security/              # (4) chief-security-officer, pentest, security-architect, legal-compliance
│   ├── development/           # (15) code-reviewer, refactor-cleaner, doc-updater, build-error-resolver,
│   │                          #      cinematic-website-builder, software-developer, web-frontend, web-backend,
│   │                          #      mobile, desktop, mcp-server, systems-programming, python, typescript, polyglot
│   ├── devops/                # (4) devops-infra, cloud-architect, azure, observability-engineer
│   ├── testing-quality/       # (4) tdd-guide, security-reviewer, e2e-runner, test-expert
│   ├── data-backend/          # (2) database-reviewer, database-architect
│   ├── languages/             # (5) go-reviewer, go-build-resolver, kotlin-reviewer, kotlin-build-resolver, python-reviewer
│   └── specialists/           # (8) game-dev, health-wellness, office-automation, search, enterprise-operations,
│   │                          #     conversational-agent, cms, reverse-engineering experts
│
├── commands/                  # 58 slash command files across 9 categories
│   ├── core/                  # (20) learn, sessions, instinct-*, eval, checkpoint, task-handoff,
│   │                          #      wrapup, skill-create, evolve, save-session, resume-session, company, ...
│   ├── development/           # (8) code-review, build-fix, refactor-clean, verify, update-*, pm2, ...
│   ├── testing-quality/       # (4) tdd, e2e, quality-gate, test-coverage
│   ├── planning/              # (11) plan, orchestrate, model-route, loop-*, multi-*, brainstorm
│   ├── languages/             # (8) go-*, kotlin-*, gradle-build, python-review
│   ├── content/               # (3) youtube, blog, social
│   ├── marketing/             # (2) seo, campaign
│   ├── design/                # (1) ui
│   └── specialized/           # (1) claw
│
├── rules/                     # common/ + 7 language dirs
│   ├── common/                # 9 files: agents, coding-style, dev-workflow, git, hooks,
│   │                          #          patterns, performance, security, testing
│   ├── golang/                # 5 files
│   ├── kotlin/                # 5 files
│   ├── perl/                  # 5 files
│   ├── php/                   # 5 files
│   ├── python/                # 5 files
│   ├── swift/                 # 5 files
│   └── typescript/            # 5 files
│
├── contexts/                  # 6 context files: dev.md, research.md, review.md, content.md, marketing.md, design.md
├── hooks/                     # hooks.json + README.md
├── mcp-configs/mcp-servers.json  # 23 MCP server configs (tokens sanitized)
│
├── repos/                     # External reference repos
├── scripts/                   # Cross-platform hook utilities
│
├── CLAUDE.md                  # Library overview & usage guide
├── AGENTS.md                  # Agent orchestration instructions
├── BRAIN.md                   # THIS FILE
├── skills-lock.json           # 7 installed skills from anthropics/claude-code
└── run.txt
```

---

## 4. File Formats

### 4a. Skill Format (WAT)

```markdown
---
name: skill-name
description: One-line active-voice description.
---

# Skill Title

One-two sentence intro.

---

## 1. Section Name

Second-person imperative. ("Do X", "Use Y", "Never Z")

**Rule:** Bold callout for critical constraint.

---
```

**Constraints:**
- Target <=150 lines (80-120 ideal)
- Omit anything the model knows from training data
- Tables beat prose for density
- `**Rule:**` callouts for constraints
- Numbered sections with `---` separators
- YAML frontmatter: `name` + `description` only (existing kit skills)

### 4b. Agent Format

```markdown
---
name: agent-name
description: Trigger description for proactive use.
tools: Read, Grep, Glob
model: sonnet
---

# Agent Name

Brief role.

## Your Role
## Process
## Output Format
```

**Rule:** Reviewer agents never get Write/Edit tools.

### 4c. Command Format

```markdown
---
description: What this command does.
---

Brief instruction text.

## Steps
1. Step one...
```

### 4d. Rule Format

```markdown
---
paths:
  - "**/*.go"
  - "**/go.mod"
---
# Rule Title

Content...
```

Rules use `paths` frontmatter to scope by file pattern. Language rules extend `common/` rules.

---

## 5. Core Skills Deep Dive

### Always-Included Skills (every project gets these 8)

| Skill | Purpose |
|-------|---------|
| `code-writing-software-development` | Read-before-edit discipline, 6-phase verification loop |
| `continuous-learning` | Extracts instincts from sessions via hooks, scores by confidence |
| `strategic-compact` | Context compaction at phase boundaries |
| `tdd-workflow` | 80%+ coverage, Red/Green/Refactor |
| `security-review` | OWASP Top 10 on every feature touching auth/input/secrets |
| `autonomous-agents-task-automation` | Parallel execution, model routing, subagent delegation |
| `notebooklm` | Second brain — every session's knowledge queryable via NotebookLM |
| `wrapup` | End-of-session knowledge capture -> pushes to NotebookLM Brain |

### The Master Skill: `generate-claude-md`

This is the **entry point** for any new project. It:
1. Gathers project context (type, stack, hosting)
2. Selects 2-6 relevant skills from the kit
3. Creates in the project: `.claude/CLAUDE.md` (with `@` imports), `.claude/CLAUDE.planning.md`, `.claude/project-config.md`, `.spec/` planning artifacts
4. Creates directory junctions: agents, commands, rules, hooks, contexts
5. Creates `.github/copilot-instructions.md` (mirrors CLAUDE.md for GitHub Copilot)
6. Runs the 3-phase gated planning workflow: requirements -> design -> tasks
7. Generates individual task files in `.spec/tasks/`

**Hard rules:** Never save gated files without user approval. Never write into the kit directory. Never copy skill content.

---

## 6. MCP Servers Available (23)

| Server | Type | Purpose |
|--------|------|---------|
| github | npx | PRs, issues, repos |
| firecrawl | npx | Web scraping |
| supabase | npx | Database operations |
| memory | npx | Persistent memory |
| sequential-thinking | npx | Chain-of-thought |
| vercel | http | Deployments |
| railway | npx | Deployments |
| cloudflare-docs | http | CF docs search |
| cloudflare-workers-builds | http | CF Workers builds |
| cloudflare-workers-bindings | http | CF Workers bindings |
| cloudflare-observability | http | CF logs |
| clickhouse | http | Analytics queries |
| exa-web-search | npx | Web research |
| context7 | npx | Live doc lookup |
| magic | npx | Magic UI components |
| filesystem | npx | File operations |
| insaits | python3 | AI-to-AI security monitoring |
| playwright | npx | Browser automation |
| fal-ai | npx | AI media generation |
| browserbase | npx | Cloud browser sessions |
| browser-use | http | AI browser agent |
| token-optimizer | npx | 95%+ context reduction |
| confluence | npx | Confluence integration |

---

## 7. Installed Skills from Official Claude Code

From `skills-lock.json` — 7 skills installed from `anthropics/claude-code`:
- Agent Development
- Command Development
- Hook Development
- MCP Integration
- Skill Development
- Writing Hookify Rules
- frontend-design

---

## 8. Key Workflows

### Adding a New Skill
1. Create `skills/<category>/<skill-name>/SKILL.md` (WAT format)
2. Add to the skill table in `skills/core/generate-claude-md/SKILL.md`
3. Update `CLAUDE.md` Available Content table
4. Check for >50% overlap with existing — merge if found

### Adding an Agent
1. Create `agents/<category>/<agent-name>.md`
2. Use agent frontmatter: `name`, `description`, `tools`, `model`
3. Reviewer agents: Read/Grep/Glob only (no Write/Edit)

### Adding a Command
1. Create `commands/<category>/<command-name>.md`
2. Use frontmatter with `description:` field
3. Available in projects via junction

### Setting Up a New Project
1. From project dir: `/generate-claude-md`
2. Describe the project -> skill selection -> junctions created
3. 3-phase planning: requirements -> design -> tasks (each needs approval)
4. Individual task files generated in `.spec/tasks/`

### Commit Convention
`feat:`, `fix:`, `docs:`, `chore:`, `refactor:` — subjects under 70 chars.

---

## 9. Critical Rules

1. **Never write project files into the kit directory** — always confirm PROJECT_ROOT first
2. **Skills <=150 lines** (80-120 ideal) — cut lowest-value sections first
3. **Tables > prose** for density
4. **Never copy skill content** into projects — always use `@` imports
5. **Reviewer agents** never get Write/Edit tools
6. **Merge if >50% overlap** with existing skills
7. **3-phase gated planning** — never save without explicit user approval
8. **Zero-copy references only** — junctions for dirs, `@` imports for skills
9. **Community skills** need security review before installation (check for prompt injection, exfiltration, tool abuse)
10. **Keep `.claude/` minimal** — only package-manager.json, instincts, and meta-skill

---

## 10. Quick Reference — Skill Category -> Path

| Category | Kit Path | Count |
|----------|----------|-------|
| Studio (Entry Points) | `skills/_studio/` | 2 |
| Core / Meta | `skills/core/` | 5 |
| Development | `skills/development/` | 11 |
| Planning | `skills/planning/` | 5 |
| Testing & Quality | `skills/testing-quality/` | 24 |
| Data & Backend | `skills/data-backend/` | 25 |
| Data Science & ML | `skills/data-science-ml/` | 60 |
| Languages | `skills/languages/` | 41 |
| AI Platform | `skills/ai-platform/` | 9 |
| Research & Docs | `skills/research-docs/` | 29 |
| DevOps | `skills/devops/` | 20 |
| DevTools | `skills/devtools/` | 18 |
| UI & Design | `skills/ui-design/` | 49 |
| Integrations | `skills/integrations/` | 107 |
| Integrations — Scraping | `skills/integrations-scraping/` | 15 |
| Frameworks — Frontend | `skills/frameworks-frontend/` | 49 |
| Frameworks — Backend | `skills/frameworks-backend/` | 24 |
| Frameworks — Mobile | `skills/frameworks-mobile/` | 11 |
| Frameworks — Desktop | `skills/frameworks-desktop/` | 24 |
| Agents & Orchestration | `skills/agents-orchestration/` | 37 |
| Architecture | `skills/architecture/` | 18 |
| Cloud | `skills/cloud/` | 24 |
| Cloud — Azure | `skills/cloud-azure/` | 122 |
| Containers & Orchestration | `skills/containers-orchestration/` | 11 |
| Security — Offensive | `skills/security-offensive/` | 36 |
| Security — Defensive | `skills/security-defensive/` | 29 |
| SEO | `skills/seo/` | 31 |
| Marketing & Growth | `skills/marketing-growth/` | 31 |
| Product & Business | `skills/product-business/` | 23 |
| E-commerce | `skills/ecommerce/` | 7 |
| ERP — Odoo | `skills/erp-odoo/` | 24 |
| Health & Wellness | `skills/health-wellness/` | 16 |
| Legal & Compliance | `skills/legal-compliance/` | 15 |
| Fintech & Payments | `skills/fintech-payments/` | 7 |
| Blockchain & Web3 | `skills/blockchain-web3/` | 8 |
| Game Development | `skills/game-dev/` | 19 |
| Media & Video | `skills/media-video/` | 10 |
| Office & Documents | `skills/office-documents/` | 11 |
| Bots | `skills/bots/` | 7 |
| Personas | `skills/personas/` | 15 |
| Team & HR | `skills/team-hr/` | 8 |
| Scientific | `skills/scientific/` | 13 |
| Functional Programming | `skills/functional-programming/` | 15 |
| Observability | `skills/observability/` | 8 |
| Search | `skills/search/` | 5 |
| Performance | `skills/performance/` | 4 |
| CMS | `skills/cms/` | 4 |
| Domain | `skills/domain/` | 5 |
| BaaS | `skills/baas/` | 2 |
| Salesforce | `skills/salesforce/` | 1 |
| Customer Support | `skills/customer-support/` | 1 |
| Specialized | `skills/specialized/` | 112 |
| **TOTAL** | | **1,219** |

---

## 11. The generate-claude-md Skill Table

This is the **routing table** — when adding any new skill, it MUST be added here or it will never be selected for projects. Located at `skills/core/generate-claude-md/SKILL.md`, Step 3.

Format:
```markdown
### Category Name
| If the project involves... | Include skill |
|---|---|
| Description of when to use | `skill-name` |
```

Current categories in the table: Core/Meta, Development, Planning & Architecture, Testing & Quality, Data & Backend, Data Science & ML, Languages, Claude/AI Platform, Research & Docs, DevOps & Infrastructure, DevTools, UI & Design, Integrations, Frameworks (Frontend/Backend/Mobile/Desktop), Agents & Orchestration, Architecture, Cloud, Cloud Azure, Containers & Orchestration, Security (Offensive/Defensive), SEO, Marketing & Growth, Product & Business, E-commerce, ERP Odoo, Health & Wellness, Legal & Compliance, Fintech & Payments, Blockchain & Web3, Game Dev, Media & Video, Office Documents, Bots, Personas, Team & HR, Scientific, Functional Programming, Observability, Search, Performance, CMS, Domain, BaaS, Specialized.

---

## 12. File Counts Summary

| Component | Count | Status |
|-----------|-------|--------|
| Skills | 1,219 | OK |
| Skill categories | 52 | OK (added `_studio/`) |
| Agents | 85 | OK (13 categories — full department coverage) |
| Commands | 58 | OK (added content/, marketing/, design/) |
| Rules (common) | 9 | OK |
| Rules (language) | 35 | OK (7 languages x 5 files) |
| Contexts | 6 | OK (added content, marketing, design) |
| Hooks | 2 | OK |
| MCP configs | 1 (23 servers) | OK |
