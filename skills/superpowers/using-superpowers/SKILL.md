---
name: using-superpowers
description: Use at the start of a task to select relevant Superpowers workflows and apply them with Codex tools.
---

# Using Superpowers in Codex

If you were dispatched as a subagent for a specific task, follow that
task's instructions without repeating this startup workflow.

Read skills the user names and skills that materially help the task.
Resolve each skill through the catalog and read its `SKILL.md` with the
available file tool. Announce the chosen skill briefly on first use.

Use process skills before implementation skills when both apply:

- New or unclear behavior: `brainstorming`, then the relevant development workflow.
- A bug or unexpected result: `systematic-debugging`, then a focused fix.
- Completion claims: `verification-before-completion`.

Read [Codex conventions](references/codex-tools.md) when applying these
workflows in Codex. Skills are guidance within the current task: system
and developer instructions, the user's scope, and existing authorization
take precedence. Scale the workflow to the change and continue work the
user has already authorized.
