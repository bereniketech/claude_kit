---
name: branch-completion
description: Guide completion of development work by presenting structured options for merge, PR, or cleanup. Use when implementation is complete and all tests pass to decide how to integrate the work.
---

# Finishing a Development Branch

---

## 1. Overview

Guide completion of development work by presenting clear options and handling the chosen workflow.

**Rule:** Verify tests → Present options → Execute choice → Clean up.

---

## 2. Verify Tests

**Before presenting options, verify tests pass:**

```bash
# Run project's test suite
npm test / cargo test / pytest / go test ./...
```

If tests fail, report failures and stop. Do not proceed to options until tests pass.

---

## 3. Determine Base Branch

```bash
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

Or ask: "This branch split from main — is that correct?"

---

## 4. Present Options

Present exactly these 4 options:

```
Implementation complete. What would you like to do?

1. Merge back to <base-branch> locally
2. Push and create a Pull Request
3. Keep the branch as-is (I'll handle it later)
4. Discard this work

Which option?
```

**Rule:** Don't add explanation — keep options concise.

---

## 5. Execute Choice

### Option 1: Merge Locally

```bash
git checkout <base-branch>
git pull
git merge <feature-branch>
<test command>   # Verify tests on merged result
git branch -d <feature-branch>
```

Then: Cleanup worktree (Step 6)

### Option 2: Push and Create PR

```bash
git push -u origin <feature-branch>
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<2-3 bullets of what changed>

## Test Plan
- [ ] <verification steps>
EOF
)"
```

Then: Cleanup worktree (Step 6)

### Option 3: Keep As-Is

Report: "Keeping branch <name>. Worktree preserved at <path>."

Do NOT cleanup worktree.

### Option 4: Discard

**Rule:** Confirm first. Require typed "discard" confirmation before deleting work.

```
This will permanently delete:
- Branch <name>
- All commits: <commit-list>
- Worktree at <path>

Type 'discard' to confirm.
```

If confirmed:
```bash
git checkout <base-branch>
git branch -D <feature-branch>
```

Then: Cleanup worktree (Step 6)

---

## 6. Cleanup Worktree

**For Options 1, 2, 4:**

Check if in worktree:
```bash
git worktree list | grep $(git branch --show-current)
```

If yes:
```bash
git worktree remove <worktree-path>
```

**For Option 3:** Keep worktree.

---

## 7. Quick Reference

| Option | Merge | Push | Keep Worktree | Cleanup Branch |
|--------|-------|------|---------------|----------------|
| 1. Merge locally | Yes | - | - | Yes |
| 2. Create PR | - | Yes | Yes | - |
| 3. Keep as-is | - | - | Yes | - |
| 4. Discard | - | - | - | Yes (force) |

---

## 8. Red Flags

**Never:**
- Proceed with failing tests
- Merge without verifying tests on the result
- Delete work without confirmation
- Force-push without explicit request

**Always:**
- Verify tests before offering options
- Present exactly 4 options
- Get typed confirmation for Option 4
- Clean up worktree for Options 1 & 4 only
