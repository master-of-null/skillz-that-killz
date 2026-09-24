# Codex tools and conventions

Use the tools and schemas available in the current session. Codex app and
CLI versions can expose different capabilities; this reference does not
grant tools, permissions, or configuration settings.

## Skills and instructions

- Personal skills live under `~/.agents/skills/`. This collection groups
  Superpowers under `~/.agents/skills/superpowers/<skill>/SKILL.md`.
- Read a relevant skill from its catalog path using the available file
  tool. Codex does not require a Claude `Skill` tool or `@` file syntax.
- The `superpowers` folder organizes files; it does not add a plugin
  namespace. Use the frontmatter name, such as `$brainstorming`, and
  resolve references through the skill catalog.
- Global instructions are loaded from `$CODEX_HOME/AGENTS.md` (normally
  `~/.codex/AGENTS.md`), which can link to `~/.agents/AGENTS.md`.
- Use a plan tool when available and useful, or a short Markdown
  checklist. Never create a persistent goal or a new user-owned task
  merely to track an ordinary checklist.

## Delegation

- Use native subagents for concrete, bounded work that can run alongside
  useful local work. Assign separate files for concurrent edits.
- With `collaboration.spawn_agent`, use `task_name` and `message`.
  `fork_turns: "none"` gives a focused child only the context you provide.
  Omit model overrides to inherit the parent's settings by default.
- When an explicit model choice is appropriate, check the current
  allowlist and schema. In the current collaboration interface, overrides
  require `fork_turns: "none"` or a positive number; `"all"` inherits the
  parent settings and rejects model and reasoning overrides. Do not pass
  `agent_type` unless the actual tool schema exposes it.
- Use `send_message` for a running child and `followup_task` to give an
  idle child more work. Use only lifecycle tools present in the session.
- Continue independent work while children run. When idle, wait using the
  available event tool within the session's time limits; avoid repeated
  immediate polls and keep the user informed while work continues.
- New Codex tasks are user-owned. Create one only when the user asks for
  a separate task; they are not a substitute for subagent tools.

## Authorization and workspace

The user's request and prior approval carry forward. State your approach
and continue authorized, reversible work. Ask only for a material missing
decision or an action outside the authorized scope. Follow actual
sandbox approval results; do not invent additional permission gates.

Read the repository state before changing branches or creating worktrees:

```bash
git rev-parse --show-toplevel
git rev-parse --absolute-git-dir
git rev-parse --path-format=absolute --git-common-dir
git branch --show-current
git status --short
git worktree list --porcelain
```

Reuse an existing isolated workspace. An empty branch name means detached
HEAD, not a sandbox restriction: branch creation may still be possible.
Only remove a worktree when this task is known to own it and its contents
are safe to remove. Its directory name alone does not establish ownership.

Verify the changed behavior with appropriate checks. Reuse a passing
result for an unchanged tree; run again after relevant changes. Report
what was checked and any actual limitation.
