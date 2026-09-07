# simplify (archived)

This project is no longer maintained. [Claude Code v2.1.154](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md#21154) restored a dedicated `/simplify` cleanup workflow, so this standalone skill is no longer needed. Use the built-in command instead.

## Migration

1. Remove this repository's installed `~/.claude/skills/simplify/SKILL.md`. Save any personal modifications before removing it. If you installed the skill elsewhere, remove that copy too.
2. Start a new Claude Code session and use the built-in `/simplify`.
3. To request Sonnet review agents, run:

   ```text
   /simplify use sonnet agents
   ```

   For a persistent preference, add this rule to your user or project `CLAUDE.md`:

   ```markdown
   - When running `/simplify`, use Sonnet for every review subagent by explicitly setting `model: "sonnet"` on each Agent call.
   ```

## Historical implementation

This skill provided parallel code review and cleanup when Claude Code's bundled `/simplify` was removed, with review subagents pinned to Sonnet. The original [SKILL.md](./SKILL.md) remains available for reference, but will receive no further updates.

## License

MIT. See [LICENSE](./LICENSE).
