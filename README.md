# Personal Codex skills

`~/.agents` is the editable source of truth. Codex reads the skills here
directly; RuleSync and a generation step are no longer needed.

## Layout

- `skills/<name>/SKILL.md`: personal skills.
- `skills/superpowers/<name>/SKILL.md`: the 14 locally adapted Superpowers skills.
- `AGENTS.md`: personal instructions, linked from `~/.codex/AGENTS.md`.
- `commands/`: existing custom prompts, linked from `~/.codex/prompts/`.
- `templates/`: templates used by the brief and spec prompts.

Edit these files directly. The repository and its Git history were moved
from `~/agentic_0`; no commit or push is needed for local edits to load.
Restart Codex or start a fresh task if an existing task still has old paths.
Codex-managed system skills under `~/.codex/skills/.system/` stay separate.

The Superpowers directory is a filesystem grouping, not a plugin
namespace. Skill names stay unchanged: use `$brainstorming`, for example.
Do not place a `SKILL.md` directly in `skills/superpowers/`.

The local Superpowers copy is maintained for Codex. Preserve these changes
when importing upstream updates; do not enable a second copy of the same
skills. Historical cross-platform reference files are retained as upstream
background; the active entrypoint uses `references/codex-tools.md`.

`brief` and `spec` remain custom prompts, with their templates here. Use
`/prompts:brief` or `/prompts:spec` in a Codex surface that supports custom
prompts, or ask Codex to read the corresponding command file.

RuleSync is useful if several runtimes need different generated output.
For this Codex-only setup, direct files and the three runtime symlinks
serve that purpose without synchronization. Existing Claude files were
left untouched and will no longer be synchronized.

References: [Codex skills](https://learn.chatgpt.com/docs/build-skills),
[global instructions](https://learn.chatgpt.com/docs/agent-configuration/agents-md).
