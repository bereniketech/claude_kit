---
name: skill-stocktake
description: Audit all Claude skills for quality, gaps, and overlap. Supports Quick Scan (changed files only) and Full Stocktake with subagent batch evaluation.
---

# Skill Stocktake

Audit Claude skills and commands using a quality checklist plus AI holistic judgment. Run as `/skill-stocktake`. Two modes: Quick Scan for recently changed skills, Full Stocktake for a complete review.

---

## 1. Determine Scope and Mode

Scan these paths relative to the invocation directory:

| Path | Included when |
|------|--------------|
| `~/.claude/skills/` | Always |
| `{cwd}/.claude/skills/` | Directory exists |

At startup, explicitly list which paths were found and will be scanned.

Mode selection:

| Mode | Trigger | Duration |
|------|---------|---------|
| Quick Scan | `results.json` exists | 5–10 min |
| Full Stocktake | `results.json` absent, or `/skill-stocktake full` | 20–30 min |

Results cache: `~/.claude/skills/skill-stocktake/results.json`

---

## 2. Quick Scan Flow

Re-evaluate only skills changed since the last run.

1. Read `results.json`.
2. Run `bash ~/.claude/skills/skill-stocktake/scripts/quick-diff.sh results.json` — outputs changed file list.
3. If output is `[]`: report "No changes since last run." and stop.
4. Re-evaluate only changed files using the Phase 2 checklist below.
5. Carry forward unchanged skills from previous results.
6. Output only the diff, then save with `save-results.sh`.

**Rule:** For Keep verdicts on mtime-only changes, restate the original rationale — never write "unchanged" alone.

---

## 3. Full Stocktake — Phase 1: Inventory

Run `bash ~/.claude/skills/skill-stocktake/scripts/scan.sh` to enumerate skill files, extract frontmatter, and collect UTC mtimes.

Present the scan summary:
```
Scanning:
  ✓ ~/.claude/skills/         (N files)
  ✗ {cwd}/.claude/skills/    (not found — global skills only)
```

Then present an inventory table: Skill | 7d use | 30d use | Description.

---

## 4. Full Stocktake — Phase 2: Quality Evaluation

Launch a general-purpose subagent with the full inventory. Process ~20 skills per invocation. Save intermediate results (`status: "in_progress"`) after each chunk. On startup, if `status: "in_progress"` is found, resume from the first unevaluated skill.

Each skill is evaluated against this checklist:
- Content overlap with other skills checked
- Overlap with MEMORY.md / CLAUDE.md checked
- Freshness of technical references verified (use WebSearch if tool names / CLI flags / APIs are present)
- Usage frequency considered
- **Token efficiency:** flag skills >200 lines unless line count is justified by genuine complexity
- **Redundancy:** flag repeated phrases, preamble, filler, or sections that restate general model knowledge
- **WAT format adherence:** YAML frontmatter (name + description only), numbered sections, `---` separators, second-person imperative, `**Rule:**` callouts

Verdict criteria:

| Verdict | Meaning |
|---------|---------|
| Keep | Useful and current |
| Improve | Worth keeping, specific improvements needed (includes: bloated >200 lines without justification) |
| Update | Referenced technology is outdated |
| Retire | Low quality, stale, or cost-asymmetric |
| Merge into [X] | Substantial overlap with another skill |

Evaluation is holistic AI judgment — not a numeric rubric. Key dimensions: actionability, scope fit, uniqueness, currency.

**Rule:** The `reason` field must be self-contained and decision-enabling. State specific evidence, not summary labels.

Reason quality examples:
- **Retire** — bad: `"Superseded"` / good: `"disable-model-invocation: true already set; superseded by continuous-learning-v2 which covers all same patterns. No unique content remains."`
- **Merge** — bad: `"Overlaps with X"` / good: `"42-line thin content; Step 4 of chatlog-to-article already covers the same workflow. Integrate the 'article angle' tip as a note there."`
- **Improve** — bad: `"Too long"` / good: `"276 lines; Section 'Framework Comparison' (L80–140) duplicates ai-era-architecture-principles; delete it to reach ~150 lines."`

---

## 5. Full Stocktake — Phase 3: Summary Table

| Skill | 7d use | Verdict | Reason |
|-------|--------|---------|--------|

---

## 6. Full Stocktake — Phase 4: Consolidation

For **Retire / Merge**: present detailed justification before confirming with user — what specific problem was found, what alternative covers it, what impact removal has on dependent skills or MEMORY.md references.

For **Improve**: present specific improvement suggestions with rationale (what to change, why, target size if applicable). User decides whether to act. For structurally broken or excessively verbose skills, suggest running `/skill-create` to regenerate a clean baseline from git history.

For **Update**: present updated content with verified sources.

**Rule:** When suggesting improvements, always include a target line count. Prefer surgical deletions over rewrites. Never add content that duplicates what the model already knows from training data.

After all verdicts: check MEMORY.md line count and propose compression if >100 lines.

**Rule:** Archive and delete operations always require explicit user confirmation.

---

## 7. Skill Creation Integration

When a stocktake verdict is **Improve** and the skill is structurally broken or severely bloated (>300 lines), offer to regenerate it using `/skill-create`:

1. Run `/skill-create` to extract a clean git-history-derived baseline
2. Reshape the output into WAT format (see `claude-kit` skill Section 2)
3. Apply the cost-efficiency rules: ≤150 lines, actionable-only sections, tables over prose

**Rule:** `/skill-create` is a baseline tool, not a final output — always apply WAT format and cost-efficiency review before saving.

---

## 8. Results File Schema

`~/.claude/skills/skill-stocktake/results.json`:

```json
{
  "evaluated_at": "2026-02-21T10:00:00Z",
  "mode": "full",
  "batch_progress": {
    "total": 80,
    "evaluated": 80,
    "status": "completed"
  },
  "skills": {
    "skill-name": {
      "path": "~/.claude/skills/skill-name/SKILL.md",
      "verdict": "Keep",
      "reason": "Concrete, actionable, unique value for X workflow",
      "mtime": "2026-01-15T08:30:00Z"
    }
  }
}
```

**Rule:** `evaluated_at` must be actual UTC time from `date -u +%Y-%m-%dT%H:%M:%SZ` — never a date-only approximation.

---
