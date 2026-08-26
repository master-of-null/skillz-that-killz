---
name: tdd
description: >-
  Use when the user asks for test-driven development, TDD, test-first
  implementation, red/green/refactor, or wants a feature or bugfix implemented
  by writing failing tests before production code. Guides coding work through a
  focused red, green, refactor loop. Do not use for read-only reviews,
  explanations, or tiny mechanical edits unless the user explicitly asks for
  TDD.
targets:
  - '*'
---
# TDD

Use this skill as an operating mode for code changes where behavior should be
designed and protected by tests before production code changes.

## Decision Rule

Use TDD when the user explicitly asks for TDD, test-first work, red/green/refactor,
or a feature/bugfix implemented with tests first.

Do not force this skill onto read-only analysis, code review, pure documentation
edits, formatting-only edits, dependency chores, or one-line mechanical changes
unless the user asks for TDD.

## Workflow

1. Classify the work.
   - Bugfix: gather the expected behavior, actual behavior, reproduction path,
     affected surface, and any error output.
   - Feature: find the spec, ticket, acceptance criteria, or user-provided
     behavior. If none exists, ask for the smallest clarification needed.
   - Refactor: confirm the existing behavior that must remain unchanged and
     identify the tests or executable checks that already protect it.

2. Read before editing.
   - Inspect the existing implementation and the nearest tests.
   - Prefer the repo's current test framework, fixtures, naming, and file
     placement.
   - Choose the smallest test layer that proves the public behavior. Use browser
     or end-to-end tests only when that is the existing convention or the user
     explicitly asks for that level.

3. Define the intended behavior.
   - Write down the public behavior being protected: inputs, outputs, visible UI
     changes, API responses, state transitions, errors, and edge cases.
   - Keep assertions tied to user-facing or caller-facing behavior, not private
     implementation details.

4. Red phase: write the failing test first.
   - Add or update the focused test before production code.
   - Run the narrowest relevant test command.
   - Confirm the test fails for the expected reason. If it passes unexpectedly,
     tighten the test before continuing.

5. Green phase: implement the smallest coherent fix or feature.
   - Change production code only after observing the red state.
   - Follow the patterns found in the existing code.
   - Keep the public surface as small as possible.
   - Run the focused test until it passes.

6. Refactor phase: clean up while tests stay green.
   - Remove duplication, dead code, unused imports, and speculative helpers.
   - Simplify control flow when it improves readability without broadening scope.
   - Keep comments sparse and only where they clarify non-obvious behavior.
   - Re-run the focused test after refactoring.

7. Verify the surrounding impact.
   - Run related tests for the touched area.
   - Run lint, typecheck, or formatting checks when the repo provides them and the
     change risk justifies it.
   - If no automated test harness exists, create or run the smallest executable
     check available and clearly report the remaining test gap.

8. Present the result.
   - State the behavior covered, the red failure observed, the implementation
     change, and the verification commands run.
   - Mention any tests or quality gates that could not be run.

## Quality Bar

- Tests are written or updated before production code.
- The first focused test run fails for the expected reason.
- The implementation is the smallest useful change that makes the behavior pass.
- Tests exercise public behavior rather than private internals.
- Refactoring happens only after green tests.
- Verification covers the changed behavior and the nearest likely regression
  surface.

## Avoid

- Writing production code before a meaningful failing test or executable check.
- Adding broad abstractions for possible future requirements.
- Padding test files with low-value assertions just to increase coverage.
- Replacing a focused unit or integration test with a slow end-to-end test unless
  the behavior can only be proven through that path.
- Waiting for user confirmation between red and green unless the user explicitly
  asked for a checkpoint.
