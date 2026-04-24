---
name: security-scan
description: Scan Claude Code configuration files for secrets, injection risks, and misconfigurations using AgentShield.
---

# Security Scan

Audit the `.claude/` directory for vulnerabilities before committing configuration changes or onboarding to a new repository.

---

## 1. When to Run

Run a scan when setting up a new project, after modifying `CLAUDE.md`, `settings.json`, or MCP configs, before any commit touching `.claude/`, and on a regular cadence as a hygiene check.

**Rule:** Never push `.claude/` changes to a shared repository without a passing scan.

---

## 2. Install AgentShield

Check for an existing installation before installing. Prefer global install for repeated use; use `npx` for one-off scans.

```bash
npx ecc-agentshield --version          # check if installed
npm install -g ecc-agentshield         # global install
npx ecc-agentshield scan .             # one-off via npx
```

**Rule:** Always verify the installed version matches the expected release before trusting scan results.

---

## 3. Run a Basic Scan

Scan the current project with default settings. Filter by severity to reduce noise in early-stage projects.

```bash
npx ecc-agentshield scan                          # scan .claude/ in cwd
npx ecc-agentshield scan --path /path/to/.claude  # explicit path
npx ecc-agentshield scan --min-severity medium    # filter low-severity
```

**Rule:** Always resolve Critical and High findings before merging; Medium findings must be triaged and documented.

---

## 4. Output Formats

Use terminal output for interactive review. Use JSON or Markdown for CI pipelines and automated reporting. Use HTML for shareable audit reports.

```bash
npx ecc-agentshield scan                            # terminal (default)
npx ecc-agentshield scan --format json              # CI integration
npx ecc-agentshield scan --format markdown          # docs
npx ecc-agentshield scan --format html > report.html
```

---

## 5. Auto-Fix Safe Issues

Apply auto-fixable corrections with `--fix`. This replaces hardcoded secrets with environment variable references and tightens wildcard permissions. It never modifies findings that require manual judgment.

```bash
npx ecc-agentshield scan --fix
```

**Rule:** Review every auto-fix diff before committing — never blindly accept automated changes.

---

## 6. Deep Analysis with Opus

Run the adversarial three-agent pipeline for higher-confidence results on sensitive configurations. Requires `ANTHROPIC_API_KEY`.

```bash
export ANTHROPIC_API_KEY=your-key
npx ecc-agentshield scan --opus --stream
```

The pipeline runs: Attacker (red team finds vectors), Defender (blue team recommends hardening), Auditor (synthesizes final verdict).

**Rule:** Use deep analysis for any configuration that grants Bash access or manages production secrets.

---

## 7. CI Integration

Add AgentShield to the pull request pipeline to block merges on security regressions.

```yaml
- uses: affaan-m/agentshield@v1
  with:
    path: '.'
    min-severity: 'medium'
    fail-on-findings: true
```

**Rule:** Set `fail-on-findings: true` in CI so security regressions block merge automatically.

---

## 8. Interpreting Severity Grades

| Grade | Score | Action |
|-------|-------|--------|
| A | 90-100 | Secure — no action required |
| B | 75-89 | Minor issues — fix before next release |
| C | 60-74 | Needs attention — fix this sprint |
| D | 40-59 | Significant risk — fix immediately |
| F | 0-39 | Critical — block all deployments |

Critical findings to fix immediately: hardcoded API keys, `Bash(*)` in allow lists, command injection via hook interpolation, shell-running MCP servers.

**Rule:** A grade below B must be resolved before any production deployment proceeds.
