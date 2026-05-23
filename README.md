# simplify

A Skill that attempts to replicates the original bundled `/simplify` command that was removed in Claude Code v2.1.146. It's code review and cleanup on the changes you've made in your current branch. 

## What it does

Captures your current diff (`git diff HEAD` + untracked files), then runs **three review subagents in parallel** — each focused on a different dimension:

1. **Code Reuse** — missed opportunities to use existing helpers, hand-rolled logic that duplicates utilities elsewhere in the repo
2. **Code Quality** — redundant state, parameter sprawl, copy-paste, stringly-typed code, verbose patterns
3. **Efficiency** — unnecessary work, N+1 patterns, missed concurrency, hot-path bloat

Reviewers **do not edit files**. They return their findings as JSON. The main agent then merges everything, resolves overlaps deterministically using cross-cutting principles, and applies edits sequentially.

### Principles (applied to every finding)

- **Preserve behavior** — a refactor that changes output on any input (even malformed input) is a redesign, not a simplification
- **Clarity over brevity** — fewer lines is not the goal; a reader grokking the code on first pass is
- **Respect house conventions** — `CLAUDE.md` rules win over generic idioms

When two reviewers propose different rewrites for the same region, the principles break the tie. When principles can't discriminate, **neither is applied** — both are logged under "Noticed but not applied" so you can opt in if you want one.

### Why review-then-apply (and not edit-in-parallel)

If three subagents call `Edit` on the same files concurrently, they race. Late writers either fail loudly or — worse — silently skip when they see "my change is already there", dropping real findings. Parallelizing the slow part (analysis) while serializing writes gets both speed and correctness.

## Install

Drop the skill into your global skills directory:

```bash
mkdir -p ~/.claude/skills/simplify
curl -fsSL https://raw.githubusercontent.com/gtapps/simplify/main/SKILL.md \
  -o ~/.claude/skills/simplify/SKILL.md
```

Or clone and copy:

```bash
git clone https://github.com/gtapps/simplify.git
cp simplify/SKILL.md ~/.claude/skills/simplify/SKILL.md
```

Then invoke with `/simplify` in any Claude Code session.

## Usage

```
/simplify
/simplify memory efficiency
/simplify "avoid breaking the public API"
```

The optional argument is passed to all three reviewers as a focus hint — they'll weight that dimension when triaging findings.

## Output

After applying, the skill prints a per-file summary plus a "Noticed but not applied" section and totals:

```
path/to/file.ts
  ✓ [Quality] removed `=== true` comparison on line 31
  ✓ [Reuse]   replaced manual loop with `.reduce(...)` on line 53
  ⊘ [Efficiency] skipped — old_string no longer matches (subsumed)

Noticed but not applied:
  ⚠ [Reuse] proposed `email.partition("@")` with None return for no-@ inputs
      (lines 95-99) — behavior change vs original (`""`). To apply: ask explicitly.

Totals: applied 2 · deduped 1 · principle-rejected 1 · stale-anchor skips 1 · parse failures 0
```

## Cost / requirements

- Subagents are pinned to `claude-sonnet-4-6` for predictable cost regardless of the parent session's model
- Typical run: 3 Sonnet calls (one per reviewer) plus the main-agent merge + apply pass
- For tiny diffs (<~20 lines, single concern), the skill skips Phase 2 and dispatches a single reviewer to avoid noise
- The skill is **non-interactive** — it never stops mid-run to ask. Rejected proposals end up in the report, not in a prompt

## License

MIT — see [LICENSE](./LICENSE).
