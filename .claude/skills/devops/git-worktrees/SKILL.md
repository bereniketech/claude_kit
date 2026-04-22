---
name: git-worktrees
description: Create isolated git worktrees for feature work with smart directory selection and safety verification. Use when starting feature work that needs isolation from current workspace or before executing implementation plans.
---

# Git Worktrees

---

## 1. Overview

Git worktrees create isolated workspaces sharing the same repository, allowing work on multiple branches simultaneously without switching.

**Rule:** Systematic directory selection + safety verification = reliable isolation.

---

## 2. Directory Selection

Follow this priority order:

### Check Existing Directories

```bash
ls -d .worktrees 2>/dev/null     # Preferred (hidden)
ls -d worktrees 2>/dev/null      # Alternative
```

If found, use that directory. If both exist, `.worktrees` wins.

### Check CLAUDE.md

```bash
grep -i "worktree.*director" CLAUDE.md 2>/dev/null
```

If preference specified, use it without asking.

### Ask User

If no directory exists and no CLAUDE.md preference:

```
No worktree directory found. Where should I create worktrees?

1. .worktrees/ (project-local, hidden)
2. A custom global location

Which would you prefer?
```

---

## 3. Safety Verification

### For Project-Local Directories (.worktrees or worktrees)

**Rule:** Verify directory is ignored before creating worktree.

```bash
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

If NOT ignored: add appropriate line to .gitignore, commit the change, then proceed. This prevents accidentally committing worktree contents to the repository.

### For Global Directories

No .gitignore verification needed — outside project entirely.

---

## 4. Creation Steps

### Detect Project Name

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
```

### Create Worktree

```bash
# Create worktree with new branch
git worktree add "$LOCATION/$BRANCH_NAME" -b "$BRANCH_NAME"
cd "$LOCATION/$BRANCH_NAME"
```

### Run Project Setup

Auto-detect and run appropriate setup:

```bash
# Node.js
if [ -f package.json ]; then npm install; fi
# Rust
if [ -f Cargo.toml ]; then cargo build; fi
# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi
# Go
if [ -f go.mod ]; then go mod download; fi
```

### Verify Clean Baseline

Run tests to ensure worktree starts clean:

```bash
npm test / cargo test / pytest / go test ./...
```

If tests fail, report failures and ask whether to proceed or investigate.
If tests pass, report ready.

### Report Location

```
Worktree ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

---

## 5. Quick Reference

| Situation | Action |
|-----------|--------|
| `.worktrees/` exists | Use it (verify ignored) |
| `worktrees/` exists | Use it (verify ignored) |
| Both exist | Use `.worktrees/` |
| Neither exists | Check CLAUDE.md → Ask user |
| Directory not ignored | Add to .gitignore + commit |
| Tests fail during baseline | Report failures + ask |
| No package.json/Cargo.toml | Skip dependency install |

---

## 6. Common Mistakes

- **Skipping ignore verification** — Worktree contents get tracked, pollute git status. Always use `git check-ignore` before creating project-local worktree.
- **Assuming directory location** — Creates inconsistency, violates project conventions. Follow priority: existing > CLAUDE.md > ask.
- **Proceeding with failing tests** — Can't distinguish new bugs from pre-existing issues. Report failures, get explicit permission to proceed.
- **Hardcoding setup commands** — Breaks on projects using different tools. Auto-detect from project files.

---

## 7. Red Flags

**Never:**
- Create worktree without verifying it's ignored (project-local)
- Skip baseline test verification
- Proceed with failing tests without asking
- Assume directory location when ambiguous

**Always:**
- Follow directory priority: existing > CLAUDE.md > ask
- Verify directory is ignored for project-local
- Auto-detect and run project setup
- Verify clean test baseline
