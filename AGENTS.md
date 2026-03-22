# claude_kit — Agent Instructions

This is a **distributable AI coding skill library** providing 18 specialized agents, 53 skills, 49 commands, and automated hook workflows for software development. It is a pure library — content here is installed into other projects, not used directly in this repo.

## Core Principles

1. **Agent-First** — Delegate to specialized agents for domain tasks
2. **Test-Driven** — Write tests before implementation, 80%+ coverage required
3. **Security-First** — Never compromise on security; validate all inputs
4. **Immutability** — Always create new objects, never mutate existing ones
5. **Plan Before Execute** — Plan complex features before writing code

## Available Agents

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| planner | Implementation planning | Complex features, refactoring |
| architect | System design and scalability | Architectural decisions |
| tdd-guide | Test-driven development | New features, bug fixes |
| code-reviewer | Code quality and maintainability | After writing/modifying code |
| security-reviewer | Vulnerability detection | Before commits, sensitive code |
| build-error-resolver | Fix build/type errors | When build fails |
| e2e-runner | End-to-end Playwright testing | Critical user flows |
| refactor-cleaner | Dead code cleanup | Code maintenance |
| doc-updater | Documentation and codemaps | Updating docs |
| go-reviewer | Go code review | Go projects |
| go-build-resolver | Go build errors | Go build failures |
| kotlin-reviewer | Kotlin code review | Kotlin/Android projects |
| kotlin-build-resolver | Kotlin build errors | Kotlin build failures |
| database-reviewer | PostgreSQL/Supabase specialist | Schema design, query optimization |
| python-reviewer | Python code review | Python projects |
| chief-of-staff | Communication triage and drafts | Multi-channel email, Slack, LINE |
| loop-operator | Autonomous loop execution | Run loops safely, monitor stalls |
| harness-optimizer | Harness config tuning | Reliability, cost, throughput |

## Agent Orchestration

Use agents proactively without user prompt:
- Complex feature requests → **planner**
- Code just written/modified → **code-reviewer**
- Bug fix or new feature → **tdd-guide**
- Architectural decision → **architect**
- Security-sensitive code → **security-reviewer**
- Autonomous loops / loop monitoring → **loop-operator**
- Harness config reliability and cost → **harness-optimizer**

Use parallel execution for independent operations — launch multiple agents simultaneously.

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
agents/          — 18 specialized subagents (core/, development/, testing-quality/, data-backend/, languages/)
skills/          — 53+ workflow skills organized by category
commands/        — 49 slash commands (core/, development/, testing-quality/, planning/, languages/, specialized/)
hooks/           — hooks.json trigger-based automations
rules/           — Always-follow guidelines (common/ + per-language)
scripts/         — Cross-platform Node.js hook utilities
mcp-configs/     — MCP server configurations
contexts/        — dev.md, research.md, review.md context files
```

## When Working in This Repo

- **Adding a skill:** Follow WAT format in `.claude/skills/claude-kit/SKILL.md`. Update `skills/core/generate-claude-md/SKILL.md` skill table and `CLAUDE.md`.
- **Adding an agent:** Place in `agents/<category>/`. Reviewer agents never get Write/Edit tools.
- **Adding a command:** Place in `commands/<category>/`. Use frontmatter with `description:` field.
- **Commit convention:** `feat:`, `fix:`, `docs:`, `chore:`, `refactor:` — subjects under 70 chars.

## Success Metrics

- All tests pass with 80%+ coverage
- No security vulnerabilities
- Code is readable and maintainable
- User requirements are met
