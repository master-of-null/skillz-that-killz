# Agentic

Home-level source of truth for personal agentic assets shared across AI tools.

## Global Assets

Edit source assets under:

```text
.rulesync/skills/
.rulesync/commands/
.rulesync/subagents/
.rulesync/templates/
.rulesync/rules/
```

Then publish them to Claude Code and Codex CLI with:

```bash
./scripts/sync-skills.sh
```

The generated/runtime targets are:

```text
~/.claude/CLAUDE.md
~/.claude/skills/
~/.claude/commands/
~/.claude/agents/
~/.codex/AGENTS.md
~/.codex/skills/
~/.codex/prompts/
```

Do not edit generated runtime copies directly. Codex-managed system skills under
`~/.codex/skills/.system/` are not part of this repo and should be left alone.
