---
name: yagni
description: Keep software changes simple, conventional, and limited to demonstrated requirements. Use when designing, implementing, reviewing, debugging, or refactoring code, especially when a solution is accumulating speculative features, options, modes, fallbacks, compatibility layers, wrappers, or custom infrastructure.
---

# YAGNI

Build the smallest solution that satisfies the proven requirement.

## Principles

- Implement what is needed now. Do not build for imagined future cases.
- Prefer deleting code to adding machinery.
- Use native platform behavior, standard protocols, existing project patterns, and battle-tested library APIs.
- Do not reimplement built-in capabilities.
- Keep one proven path. Avoid options, modes, fallbacks, legacy support, and compatibility layers unless a real requirement or observed failure demands them.
- Let impossible states fail plainly. Do not add defensive behavior for situations that should not occur.
- Add abstractions only when they remove demonstrated complexity or meaningful duplication.
- Treat awkward code as evidence. If the solution needs wrappers, repair ladders, or repeated patches, stop and reassess assumptions before adding more.
- When an existing abstraction is demonstrably broken, drop to the simplest correct underlying primitive. Do not build another framework around the defect.
- Prefer boring code whose behavior, lifetime, and failure modes are obvious.

## Ways to implement the principles

1. Strongly typed code will result in fewer conditions.
  - Aggressively avoid `unknown` or `any`.
  - Validate at the boundary layer. Do not let unknown types infect the code.
2. Code shall serve the application, not the tests. If application code is only used by a test, delete it.

## Working Rule

Before adding anything, ask:

1. What observed requirement needs this?
2. What existing capability already handles it?
3. What can be removed?
4. Is complexity revealing a wrong assumption?

If there is no concrete answer, do not add it.
