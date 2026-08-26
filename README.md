# Agentic

Personal source of truth for rules, skills, and commands shared across Claude Code and Codex.

## Generate

Edit the source files under `.rulesync/`, then run this from the repository root:

```bash
npx rulesync generate
```

RuleSync reads `rulesync.jsonc` and writes user-level files because `global` is enabled.

## Runtime targets

| Asset | Claude Code | Codex |
| --- | --- | --- |
| Rules | `~/.claude/CLAUDE.md` | `~/.codex/AGENTS.md` |
| Skills | `~/.claude/skills/` | `~/.agents/skills/` |
| Commands | `~/.claude/commands/` | `~/.codex/prompts/` |
| Subagents, when present | `~/.claude/agents/` | `~/.codex/agents/` |

Templates under `.rulesync/templates/` stay in this repository and support the generated commands.

Do not edit generated runtime files directly. Codex-managed system skills under
`~/.codex/skills/.system/` are not part of this repository and should be left alone.
