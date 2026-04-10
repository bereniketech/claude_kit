# claude_kit — Agent Instructions

This is a **distributable AI coding skill library** providing 82 specialized agents (organized as a holding company OS — board + 3 operating companies), 1,219 skills, 58 commands, and automated hook workflows for software development. It is a pure library — content here is installed into other projects, not used directly in this repo.

## Core Principles

1. **Agent-First** — Delegate to specialized agents for domain tasks
2. **Test-Driven** — Write tests before implementation, 80%+ coverage required
3. **Security-First** — Never compromise on security; validate all inputs
4. **Immutability** — Always create new objects, never mutate existing ones
5. **Plan Before Execute** — Plan complex features before writing code

## Available Agents

The agent suite is organized as a **holding company OS**: a board of cross-company governance roles above three operating companies, each with its own CEO and internal cascade.

### Board (4) — `agents/board/`
| Agent | Purpose |
|---|---|
| `company-coo` | Master router across all 3 operating companies — entry point for multi-company initiatives |
| `chief-of-staff` | Cross-company ops, communication triage, escalations |
| `chief-design-officer` | Cross-company design coherence (software UI ↔ marketing brand ↔ media visuals) |
| `people-operations-expert` | Corporate HR — hiring, comp, contracts, handbook |

### software-company (60) — `agents/software-company/` — CEO: `software-cto`

Internal divisions: `engineering/` (17), `ai/` (5, sub-lead `ai-cto`), `devops/` (4), `data/` (2), `qa/` (4), `languages/` (5), `product/` (10, sub-lead `chief-product-officer`), `design/` (1), `security/` (4, sub-lead `chief-security-officer`), `specialists/` (7).

Key direct-use agents: `planner`, `architect`, `code-reviewer`, `refactor-cleaner`, `doc-updater`, `build-error-resolver`, `web-frontend-expert`, `web-backend-expert`, `mobile-expert`, `desktop-expert`, `mcp-server-expert`, `python-expert`, `typescript-expert`, `polyglot-expert`, `systems-programming-expert`, `database-architect`, `database-reviewer`, `tdd-guide`, `test-expert`, `e2e-runner`, `security-reviewer`, `pentest-expert`, `security-architect`, `legal-compliance-expert`, `ui-design-expert`, `product-manager-expert`, `devops-infra-expert`, `cloud-architect`, `azure-expert`, `observability-engineer`, `go-reviewer`, `kotlin-reviewer`, `python-reviewer`, plus go/kotlin build resolvers and 7 niche specialists (game-dev, office-automation, search, enterprise-operations, conversational-agent, cms, reverse-engineering).

### marketing-company (7) — `agents/marketing-company/` — CEO: `chief-marketing-officer`

`seo-expert`, `growth-marketing-expert`, `competitor-intelligence-expert`, `paid-ads-expert`, `email-marketing-expert`, `brand-expert`.

### media-company (11) — `agents/media-company/` — CEO: `chief-content-officer`

`youtube-content-expert`, `video-production-expert`, `podcast-expert`, `blog-writing-expert`, `newsletter-expert`, `technical-writer-expert`, `image-creation-expert`, `presentation-expert`, `social-media-expert`, `content-repurposing-expert`.

## Agent Orchestration

Use agents proactively without user prompt:
- Multi-company initiatives → **company-coo**
- Anything inside software-company → **software-cto** (will route internally)
- Anything inside marketing-company → **chief-marketing-officer**
- Anything inside media-company → **chief-content-officer**
- Single direct tasks: bypass the CEO and call the specialist directly when scope is clear (e.g. `code-reviewer` after writing code, `tdd-guide` for new features, `architect` for design decisions)

Do **not** reach past a CEO into their internal org chart for multi-step work — let the CEO route. Use parallel execution for independent operations.

## Security Guidelines

**Before ANY commit:**
- No hardcoded secrets (API keys, passwords, tokens)
- All user inputs validated
- SQL injection prevention (parameterized queries)
- XSS prevention (sanitized HTML)
- Authentication/authorization verified
- Error messages don't leak sensitive data

**Secret management:** NEVER hardcode secrets. Use environment variables or a secret manager. Rotate any exposed secrets immediately.

**If security issue found:** STOP → use security-reviewer agent → fix CRITICAL issues → rotate exposed secrets → review codebase for similar issues.

## Coding Style

**Immutability (CRITICAL):** Always create new objects, never mutate. Return new copies with changes applied.

**File organization:** Many small files over few large ones. 200–400 lines typical, 800 max. Organize by feature/domain, not by type.

**Error handling:** Handle errors at every level. Log detailed context server-side. Never silently swallow errors.

**Code quality checklist:**
- Functions small (<50 lines), files focused (<800 lines)
- No deep nesting (>4 levels)
- Proper error handling, no hardcoded values

## Testing Requirements

**Minimum coverage: 80%**

**TDD workflow (mandatory):**
1. Write test first (RED) — test should FAIL
2. Write minimal implementation (GREEN) — test should PASS
3. Refactor (IMPROVE) — verify coverage 80%+

## Development Workflow

1. **Plan** — Use planner agent, identify dependencies and risks, break into phases
2. **TDD** — Use tdd-guide agent, write tests first, implement, refactor
3. **Review** — Use code-reviewer agent immediately, address CRITICAL/HIGH issues
4. **Commit** — Conventional commits format: `feat:`, `fix:`, `refactor:`, `docs:`, `chore:`

## Git Workflow

**Commit format:** `<type>: <description>` — Types: feat, fix, refactor, docs, test, chore, perf, ci

**PR workflow:** Analyze full commit history → draft comprehensive summary → include test plan → push with `-u` flag.

## Library Structure

```
agents/          — 82 agents organized as a holding company OS:
                   board/             (4)  — corporate governance
                   software-company/  (60) — CEO: software-cto
                   marketing-company/ (7)  — CEO: chief-marketing-officer
                   media-company/     (11) — CEO: chief-content-officer
skills/          — 1,219 workflow skills organized by category
commands/        — 58 slash commands (core/, development/, testing-quality/, planning/, languages/, content/, marketing/, design/, specialized/)
hooks/           — hooks.json trigger-based automations
rules/           — Always-follow guidelines (common/ + per-language)
scripts/         — Cross-platform Node.js hook utilities
mcp-configs/     — MCP server configurations
contexts/        — dev.md, research.md, review.md, content.md, marketing.md, design.md
```

## When Working in This Repo

- **Adding a skill:** Follow WAT format in `.claude/skills/claude-kit/SKILL.md`. Update `skills/_studio/generate-claude-md/SKILL.md` skill table and `CLAUDE.md`.
- **Adding an agent:** Place in the right operating company subfolder (`agents/software-company/<division>/`, `agents/marketing-company/<division>/`, `agents/media-company/<division>/`) or `agents/board/` if it's a cross-company governance role. Reviewer agents never get Write/Edit tools.
- **Adding a command:** Place in `commands/<category>/`. Use frontmatter with `description:` field.
- **Commit convention:** `feat:`, `fix:`, `docs:`, `chore:`, `refactor:` — subjects under 70 chars.

## Success Metrics

- All tests pass with 80%+ coverage
- No security vulnerabilities
- Code is readable and maintainable
- User requirements are met
