# claude_kit BRAIN

> **Purpose:** Single-read knowledge base for the entire claude_kit repository. Read this instead of scanning the codebase. Updated: 2026-04-05.

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
├── skills/                    # 61 SKILL.md files across 14 categories
│   ├── core/                  # 6: generate-claude-md, continuous-learning, eval-harness,
│   │                          #    new-features-updates, skill-stocktake, strategic-compact
│   ├── development/           # 7: code-writing, build-website, api-design, ide-integration,
│   │                          #    content-hash-cache, systematic-debugging, branch-completion
│   ├── planning/              # 5: planning-specification, autonomous-agents, multi-model,
│   │                          #    brainstorming, parallel-agent-dispatch
│   ├── testing-quality/       # 4: tdd-workflow, security-review, security-scan, e2e-testing
│   ├── data-backend/          # 3: postgres-patterns, database-migrations, clickhouse-io
│   ├── languages/             # 8: golang, python, kotlin, java, swift, cpp, android, perl
│   ├── ai-platform/           # 2: claude-developer-platform, notebooklm
│   ├── research-docs/         # 3: research, document-writing, knowledge-management
│   ├── devops/                # 2: terminal-cli-devops, git-worktrees
│   ├── ui-design/             # 8: presentations-ui-design, ui-ux-pro-max, design, brand,
│   │                          #    design-system, banner-design, slides, ui-styling
│   ├── integrations/          # 8: x-api, fal-ai, video-editing, videodb, nutrient,
│   │                          #    visa-doc-translate, whatsapp-automation, copilot-autonomous-tasks
│   ├── domain/                # 3: logistics, trade-compliance, energy-procurement
│   ├── specialized/           # 2: nanoclaw-repl, ralphinho-rfc-pipeline
│   └── additional/            # (empty — reserved)
│
├── agents/                    # 18 agent definitions
│   ├── core/                  # planner, architect, chief-of-staff, loop-operator, harness-optimizer
│   ├── development/           # code-reviewer, refactor-cleaner, doc-updater, build-error-resolver
│   ├── testing-quality/       # tdd-guide, security-reviewer, e2e-runner
│   ├── data-backend/          # database-reviewer
│   └── languages/             # go-reviewer, go-build-resolver, kotlin-reviewer, kotlin-build-resolver, python-reviewer
│
├── commands/                  # 59 slash command files
│   ├── core/                  # learn, sessions, instinct-*, eval, checkpoint, task-handoff, wrapup, ...
│   ├── development/           # code-review, build-fix, refactor-clean, verify, update-*, pm2, ...
│   ├── testing-quality/       # tdd, e2e, quality-gate, test-coverage
│   ├── planning/              # plan, orchestrate, model-route, loop-*, multi-*, brainstorm
│   ├── languages/             # go-*, kotlin-*, gradle-build, python-review
│   └── specialized/           # claw
│
├── rules/                     # common/ + 8 language dirs
│   ├── common/                # 9 files: agents, coding-style, dev-workflow, git, hooks, patterns, perf, security, testing
│   ├── golang/                # 5 files: coding-style, hooks, patterns, security, testing
│   ├── kotlin/                # 5 files
│   ├── perl/                  # 5 files
│   ├── php/                   # 5 files
│   ├── python/                # 5 files
│   ├── swift/                 # 5 files
│   └── typescript/            # 5 files
│
├── contexts/                  # 3 context files: dev.md, research.md, review.md
├── hooks/                     # hooks.json + README.md
├── mcp-configs/mcp-servers.json  # 22 MCP server configs (tokens sanitized)
│
├── new skills/                # 1,337 community skills to be triaged (see Section 8)
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
- Target ≤150 lines (80-120 ideal)
- Omit anything the model knows from training data
- Tables beat prose for density
- `**Rule:**` callouts for constraints
- Numbered sections with `---` separators
- YAML frontmatter: `name` + `description` only (existing kit skills)

**Community/new skills** have extended frontmatter: `risk`, `source`, `date_added`, `author`, `tags`, `tools`.

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
| `wrapup` | End-of-session knowledge capture → pushes to NotebookLM Brain |

### The Master Skill: `generate-claude-md`

This is the **entry point** for any new project. It:
1. Gathers project context (type, stack, hosting)
2. Selects 2-6 relevant skills from the kit
3. Creates in the project: `.claude/CLAUDE.md` (with `@` imports), `.claude/CLAUDE.planning.md`, `.claude/project-config.md`, `.spec/` planning artifacts
4. Creates directory junctions: agents, commands, rules, hooks, contexts
5. Creates `.github/copilot-instructions.md` (mirrors CLAUDE.md for GitHub Copilot)
6. Runs the 3-phase gated planning workflow: requirements → design → tasks
7. Generates individual task files in `.spec/tasks/`

**Hard rules:** Never save gated files without user approval. Never write into the kit directory. Never copy skill content.

---

## 6. MCP Servers Available (22)

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

## 8. New Skills Triage (1,337 skills in `new skills/`)

### Category Breakdown

| Category | Count | Examples | Merge Target / Action |
|----------|-------|---------|----------------------|
| **Cloud — Azure** | 116 | azure-ai-projects-py, azure-cosmos-db-py, azure-identity-dotnet | **NEW category:** `skills/cloud/azure/` — group by service |
| **UI/Design/Frontend** | 114 | react-best-practices, nextjs-app-router, angular, svelte, tailwind, shadcn, threejs | **MERGE** many into existing `ui-design/`. **NEW:** `skills/languages/react-patterns/`, `skills/languages/angular-patterns/` etc. |
| **Security** | 114 | 007, api-security-testing, burp-suite, pentest-checklist, red-team-tactics | **MERGE** best ones into existing `security-review` + `security-scan`. Keep specialized (pentest, red-team) as new. |
| **DevOps/Infrastructure** | 101 | docker-expert, kubernetes-architect, terraform-specialist, helm-chart, github-actions | **MERGE** some into existing `terminal-cli-devops`. **NEW category:** `skills/devops/` expansion |
| **Integrations/Automation** | 82 | slack-automation, jira-automation, hubspot-automation, notion-automation (mostly Composio MCP) | **NEW category:** `skills/integrations/automation/` — Composio-based automations |
| **Testing/Quality** | 66 | playwright-skill, jest patterns, pytest, bats-testing, k6-load-testing | **MERGE** into existing `tdd-workflow` + `e2e-testing`. Keep specialized ones. |
| **Research/Docs/Writing** | 65 | wiki-architect, wiki-onboarding, documentation-templates, blog-writing | **MERGE** into existing `research-docs/`. Pick best wiki/doc skills. |
| **Marketing/SEO** | 60 | seo-content, seo-technical, growth-engine, copywriting, cold-email | **NEW category:** `skills/marketing/` |
| **AI/ML** | 53 | rag-engineer, langchain, langgraph, hugging-face-*, prompt-engineering | **MERGE** some into existing `ai-platform/`. **NEW:** `skills/ai-platform/rag/`, `skills/ai-platform/langchain/` |
| **Planning/Architecture** | 50 | ddd-strategic-design, event-sourcing, microservices-patterns, c4-architecture | **MERGE** into existing `planning/`. Keep DDD/event-sourcing as new specialized. |
| **Data/Backend** | 42 | neon-postgres, vector-database, redis, mongodb, dbt-transformation | **MERGE** into existing `data-backend/`. **NEW:** `skills/data-backend/vector-db/` |
| **Agents/Orchestration** | 26 | agent-memory-systems, agent-orchestrator, multi-agent-patterns, crewai | **MERGE** into existing `autonomous-agents-task-automation` + `parallel-agent-dispatch` |
| **Languages** | 24 | rust-pro, ruby-pro, elixir-pro, haskell-pro, dotnet-backend | **NEW skills** for missing languages: rust, ruby, elixir, dotnet, etc. |
| **Claude Code Meta** | 24 | claude-code-expert, skill-creator, hook-development, agents-md, writing-skills | **MERGE** into existing meta-skill + official installed skills |
| **Mobile** | 17 | flutter-expert, expo-deployment, react-native-architecture, ios-developer | **MERGE** into existing `android-patterns`. **NEW:** `skills/languages/flutter-patterns/`, `skills/languages/react-native-patterns/` |
| **Debugging** | 17 | debugger, error-detective, phase-gated-debugging, distributed-debugging | **MERGE** into existing `systematic-debugging` |
| **Code Quality** | 17 | clean-code, code-reviewer, code-simplifier, legacy-modernizer | **MERGE** into existing `code-writing-software-development` |
| **Odoo ERP** | 14 | odoo-module-developer, odoo-orm-expert, odoo-performance-tuner | **NEW category:** `skills/domain/odoo/` |
| **Health Domain** | 14 | nutrition-analyzer, fitness-analyzer, sleep-analyzer, fda-compliance | **NEW category:** `skills/domain/health/` |
| **Makepad** | 13 | makepad-basics, makepad-widgets, makepad-shaders, makepad-layout | **NEW category:** `skills/languages/makepad-patterns/` |
| **Git Workflows** | 11 | git-advanced-workflows, create-pr, git-hooks-automation | **MERGE** into existing `branch-completion` + `git-worktrees` |
| **Blockchain/Web3** | 11 | solidity-security, defi-protocol, nft-standards, web3-testing | **NEW category:** `skills/domain/blockchain/` |
| **Persona Agents** | 10 | warren-buffett, elon-musk, bill-gates, yann-lecun, andrej-karpathy | **NEW category:** `skills/specialized/personas/` |
| **Context/Memory** | 10 | context-manager, context-optimization, conversation-memory | **MERGE** into existing `strategic-compact` + `continuous-learning` |
| **Game Dev** | 7 | unity-developer, unreal-engine-cpp-pro, godot-4-migration, bevy-ecs | **NEW category:** `skills/languages/game-engines/` |
| **WordPress** | 1 | wordpress (comprehensive) | **NEW:** `skills/languages/wordpress-patterns/` |
| **Uncategorized** | 258 | Mixed: fal-audio, auth-patterns, bash-linux, bun-dev, clerk-auth, convex, etc. | Needs per-skill triage |

### Exact Name Duplicates (10 — must compare & merge)

| Skill Name | Exists In | Action |
|------------|-----------|--------|
| `brainstorming` | `skills/planning/` | Compare quality, merge best content |
| `e2e-testing` | `skills/testing-quality/` | Compare, likely keep existing |
| `energy-procurement` | `skills/domain/` | Compare, likely keep existing |
| `notebooklm` | `skills/ai-platform/` | Compare, keep existing (heavily integrated) |
| `python-patterns` | `skills/languages/` | Compare, merge any new patterns |
| `systematic-debugging` | `skills/development/` | Compare, merge best content |
| `tdd-workflow` | `skills/testing-quality/` | Compare, likely keep existing |
| `ui-ux-pro-max` | `skills/ui-design/` | Compare, merge best content |
| `videodb` | `skills/integrations/` | Compare, likely keep existing |
| `whatsapp-automation` | `skills/integrations/` | Compare, merge if richer |

---

## 9. Key Workflows

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
2. Describe the project → skill selection → junctions created
3. 3-phase planning: requirements → design → tasks (each needs approval)
4. Individual task files generated in `.spec/tasks/`

### Commit Convention
`feat:`, `fix:`, `docs:`, `chore:`, `refactor:` — subjects under 70 chars.

---

## 10. Critical Rules

1. **Never write project files into the kit directory** — always confirm PROJECT_ROOT first
2. **Skills ≤150 lines** (80-120 ideal) — cut lowest-value sections first
3. **Tables > prose** for density
4. **Never copy skill content** into projects — always use `@` imports
5. **Reviewer agents** never get Write/Edit tools
6. **Merge if >50% overlap** with existing skills
7. **3-phase gated planning** — never save without explicit user approval
8. **Zero-copy references only** — junctions for dirs, `@` imports for skills
9. **Community skills** need security review before installation (check for prompt injection, exfiltration, tool abuse)
10. **Keep `.claude/` minimal** — only package-manager.json, instincts, and meta-skill

---

## 11. Current State

All core infrastructure is intact:
- **Agents:** 18 agent files across 5 categories
- **Commands:** 59 command files across 6 categories
- **Rules:** 9 common + 30 language-specific (6 languages)
- **Contexts:** 3 files (dev, research, review)
- **Hooks:** hooks.json + README
- **Skills:** 61 skills across 14 categories
- The `new skills/` folder contains 1,129 community skills pending triage (208 redundant duplicates removed).

---

## 12. Priority Triage Actions for New Skills

### Phase 1: Merge High-Value New Skills
1. **Security:** Best of 007, api-security-testing, pentest-checklist → merge into security-review/security-scan
2. **React/Next.js:** react-best-practices, nextjs-app-router-patterns → new `skills/languages/react-patterns/`
3. **Docker/K8s:** docker-expert, kubernetes-architect → expand devops/
4. **RAG/LangChain:** rag-engineer, langchain-architecture → expand ai-platform/
5. **DDD/Architecture:** ddd-strategic-design, event-sourcing → expand planning/

### Phase 2: Create New Categories
1. `skills/cloud/azure/` — consolidate 116 Azure skills into 3-5 pattern skills
2. `skills/marketing/` — best of 60 marketing/SEO skills
3. `skills/domain/odoo/` — 14 Odoo skills
4. `skills/domain/health/` — select best health skills
5. `skills/domain/blockchain/` — web3/solidity skills
6. `skills/languages/rust-patterns/` — from rust-pro, rust-async-patterns
7. `skills/languages/ruby-patterns/` — from ruby-pro
8. `skills/languages/dotnet-patterns/` — from dotnet-architect, dotnet-backend

### Phase 3: Discard Low-Value / Non-English / Niche
- Portuguese-only skills (advogado-*, leiloeiro-*, etc.) — skip unless user needs
- Famous-person persona agents — move to `skills/specialized/personas/` if wanted
- Vendor-locked skills (e.g., antigravity-specific) — skip
- Near-duplicate automations (40+ Composio-based *-automation skills) — consolidate into one pattern skill

---

## 13. Quick Reference — Skill Category → Path

| Category | Kit Path | Count |
|----------|----------|-------|
| Core / Meta | `skills/core/` | 6 |
| Development | `skills/development/` | 7 |
| Planning | `skills/planning/` | 5 |
| Testing & Quality | `skills/testing-quality/` | 4 |
| Data & Backend | `skills/data-backend/` | 3 |
| Languages | `skills/languages/` | 8 |
| AI Platform | `skills/ai-platform/` | 2 |
| Research & Docs | `skills/research-docs/` | 3 |
| DevOps | `skills/devops/` | 2 |
| UI & Design | `skills/ui-design/` | 8 |
| Integrations | `skills/integrations/` | 8 |
| Domain | `skills/domain/` | 3 |
| Specialized | `skills/specialized/` | 2 |
| **TOTAL** | | **61** |

---

## 14. The generate-claude-md Skill Table

This is the **routing table** — when adding any new skill, it MUST be added here or it will never be selected for projects. Located at `skills/core/generate-claude-md/SKILL.md`, Step 3.

Format:
```markdown
### Category Name
| If the project involves... | Include skill |
|---|---|
| Description of when to use | `skill-name` |
```

Current categories in the table: Core/Meta, Development, Planning & Architecture, Testing & Quality, Data & Backend, Languages, Claude/AI Platform, Research & Docs, DevOps & Infrastructure, UI & Design, Integrations, Domain, Specialized.

---

## 15. File Counts Summary

| Component | Count | Status |
|-----------|-------|--------|
| Skills | 61 | OK |
| Agents | 18 | OK |
| Commands | 59 | OK |
| Rules (common) | 9 | OK |
| Rules (language) | 30 | OK |
| Contexts | 3 | OK |
| Hooks | 2 | OK |
| MCP configs | 1 | OK |
| New skills (to triage) | 1,129 | PENDING (208 redundant removed) |
