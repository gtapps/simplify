# simplify

A Skill based of the original bundled `/simplify` command removed in Claude Code v2.1.146 — code review and cleanup on the changes in your current branch.

> **Note:** v2.1.152 brought `/simplify` back as `/code-review --fix`. This skill is a cheaper alternative — it pins subagents to Sonnet instead of Opus like the original one did.

## What it does

Captures your current diff (`git diff HEAD` + untracked files), then runs **three review subagents in parallel**:

1. **Code Reuse** — missed opportunities to use existing helpers, logic that duplicates utilities elsewhere in the repo
2. **Code Quality** — redundant state, parameter sprawl, copy-paste, stringly-typed code, verbose patterns
3. **Efficiency** — unnecessary work, N+1 patterns, missed concurrency, hot-path bloat

Reviewers return findings as JSON. The main agent merges them, filters against behavior-preservation and clarity rules, and applies edits sequentially. Proposals that conflict or risk behavior changes are logged under "Noticed but not applied" instead of applied.

## Install

```bash
mkdir -p ~/.claude/skills/simplify
curl -fsSL https://raw.githubusercontent.com/gtapps/simplify/main/SKILL.md \
  -o ~/.claude/skills/simplify/SKILL.md
```

Then invoke with `/simplify` in any Claude Code session.

## Usage

```
/simplify
/simplify memory efficiency
/simplify "avoid breaking the public API"
```

The optional argument is passed to all three reviewers as a focus hint.

## Output

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

## License

MIT — see [LICENSE](./LICENSE).
