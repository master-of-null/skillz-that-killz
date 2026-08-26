---
name: implement-feature
description: >-
  Implement a feature from a finalized plan using test-driven interface design:
  read the codebase first, define the intended public behavior, write failing
  tests before production code, implement against the red/green/refactor loop,
  and finish with quality gates for imports, call sites, dead code, slop, and
  acceptance coverage. Use when the user says 'implement this', 'build this',
  'go implement', 'implement feature', 'run with this plan', 'ok let's do it',
  'ship it', 'go ahead and build it', or hands off a plan/spec for
  implementation. Also trigger when the user finishes a planning conversation
  and gives a green light to start coding. Do NOT use for reviews, refactors, or
  one-line changes — this is for multi-file feature work where the user has
  already decided what to build.
targets:
  - '*'
---
# Implement Feature

Turn a finalized plan into code that's correct on the first pass. The codebase is the source of truth, not your training data. Read before you write, challenge the plan against actual contracts, design the public interface through tests, match what exists, verify what you produced.

## Input

The user will provide or reference a plan. If the plan is ambiguous on a point that affects architecture, ask one round of clarifying questions. Do not guess at architectural decisions — but don't stall on minor details either.

If the plan contains a likely blocker, correct the plan before coding instead of implementing the flawed version and waiting for review to catch it. A blocker needs concrete evidence: a violated acceptance criterion, repo contract, runtime path, schema, caller expectation, or existing behavior.

## Workspace Policy

Use the current checkout by default. Do not create, switch to, or rely on git worktrees unless the user explicitly asks you to use worktrees for the task.

## Phase 1: Learn the Codebase

Before writing any production code, build a concrete understanding of the codebase you're working in. This phase exists because the #1 source of implementation bugs is assuming you know what an API looks like instead of reading it. Your training data is stale; the codebase is the source of truth.

### 1a. Find the nearest neighbors

Find and read the components most similar to what you're about to build. If building a new agent, read every existing agent. If adding a pipeline step, read every existing step.

For each, note and **write out explicitly** in your response:
- Class hierarchy and base class
- Constructor signature and how dependencies are injected
- How consumers instantiate and call it
- Naming conventions (method names, private/public, async prefixes)
- Error handling style (catch or propagate? what gets logged?)

Writing these down isn't busywork — it's forcing yourself to commit to concrete patterns before you start freelancing. The user will also see them and can correct you early if you've misread something.

### 1b. Trace the APIs you'll call

For every function, method, or class you plan to use from another module:
- **Read the actual source file** and confirm it exists
- Note the exact signature (parameter names, types, return type)
- Note the import path

LLMs routinely hallucinate plausible-looking function signatures that don't exist. `mission_spec.load_missions()` looks right but the real function might be `project.load_missions()`. The only way to catch this is to read the source — not to rely on what seems familiar from training data.

### 1c. Build the ground-truth inventory

For features that map, adapt, migrate, validate, ingest, route, render, or otherwise coordinate existing behavior, do not infer coverage from registry names, docs, or naming conventions alone. Build an inventory from the actual producers and consumers in the repo.

Write out the relevant ground truth before proposing files:
- Actual producers/emitters/callers, with the source files you inspected
- Actual subjects/routes/events/payload fields/config keys/schema variants, including missing or optional fields
- Existing consumers/readers and their expectations
- For each item, the intended treatment: map, modify, create, preserve, ignore, dead-letter, reject, or leave out of scope
- Any long-term architectural concerns that are real but not necessary for the current ticket

If you use or receive adversarial review, triage every challenge explicitly:
- `must-change`: the implementation would be wrong, fail acceptance criteria, or break an existing contract
- `should-change`: improves correctness or test quality without expanding scope
- `record-only`: valid longer-term concern, but not required for this ticket
- `reject`: not supported by repo evidence or conflicts with the requested scope

Only expand implementation scope for `must-change` findings, or for `should-change` findings that stay inside the original acceptance criteria. Record longer-term concerns without turning them into code unless the user explicitly asks. Do not accept a `must-change` label on vibes alone; require the exact broken plan step, the violated repo fact, the concrete failure, and the smallest fix.

### 1d. Pre-implementation challenge pass

Before writing tests or production code, run a short self-review of the plan:
- Which existing contract could this violate if the plan is wrong?
- Which producer, consumer, schema variant, config key, route, or runtime path have you not inspected yet?
- Which call signature or import path are you assuming rather than reading?
- What old code, call site, config, or documentation might become orphaned?
- Which acceptance criterion would still be untested after the planned tests?
- Which file, abstraction, option, interface, config flag, or helper has **no present consumer** — no caller, test, command, UI surface, workflow, or alert that uses it the moment this lands?

If the answer exposes a likely failure inside the requested scope, update the file plan before coding. If it exposes only long-term cleanup, record it as out of scope. If it is speculative, reject it and move on.

For the YAGNI question specifically: "we might need this later," "future flexibility," "in case we extend it," and "for symmetry with X" do not count as present consumers. Drop those items from the plan or move them to the explicit out-of-scope list with a one-line reason. Robustness for behavior that exists is welcome; scaffolding for behavior you haven't seen yet is not.

### 1e. Present the file plan - CHECKPOINT

Before you write tests or production code, present this to the user and wait for their go-ahead when the plan is high-risk, ambiguous, or changes architecture. If the user already gave an explicit implementation go-ahead and the scope is clear, keep this as a concise working checkpoint and proceed:
- Files you'll **create** (and why a new file is needed vs extending an existing one)
- Files you'll **modify** (and what changes)
- Files you'll **delete** (if you're replacing something)
- The test files or verification harness you'll add/update first
- Ground-truth inventory summary for any mapped/adapted existing behavior
- Explicit out-of-scope items that are valid but not part of this implementation
- Any pre-implementation challenge findings and how you resolved, scoped, or rejected them

When waiting for approval, this is the user's last chance to course-correct before implementation. Keep it concise — a bulleted list, not an essay.

## Phase 2: Design the Interface Through Tests

Tests are executable interface design. Before writing production code, define the feature from the consumer's point of view, then encode that intended usage as failing tests or an equivalent executable check.

Write down the intended interface before implementing:
- The public call site, UI flow, command, API shape, event, or integration boundary
- Inputs, outputs, errors, state transitions, and important edge cases
- What should feel ergonomic for the caller or user
- Which existing test and implementation patterns this should match

Write the first tests before production code:
- Prefer tests that exercise public behavior rather than private helpers
- Make tests read like usage examples for the interface being designed
- Assert observable outcomes, not internal implementation details
- Cover acceptance criteria and risky edge cases; don't pad with low-value assertions
- Update existing tests first when they already describe the intended behavior

Run the focused test command and confirm the new or changed test fails for the expected reason. Do not start production implementation until you have a meaningful red state.

If automated tests are impractical, define the evaluation loop first: a script, fixture, browser flow, CLI command, snapshot, or manual verification checklist tied directly to acceptance criteria. Run it before implementation and confirm it currently fails or exposes the missing behavior.

## Phase 3: Implement Against Red Tests

Write the code. The patterns you cataloged in Phase 1 are your guardrails:

**Follow existing patterns exactly.** Same base class, same method signatures, same naming. If the codebase uses `async_call_llm`, you use `async_call_llm`. If agents inherit from `AgentBase` and override `run()`, yours does too. Deviating from established patterns creates inconsistency the user has to reconcile later — if you genuinely need to deviate, call it out explicitly and explain why.

**Write code the way the existing codebase writes code.** Match the style of the file you're editing. If it has no docstrings, don't add docstrings. If it doesn't use try/catch, don't add try/catch. If comments are rare, keep yours rare. The goal is code that looks like it was written by the same person who wrote the rest of the file — not code that looks like an AI wrote one function in the middle.

**Clean up completely.** If you create a new class or module that replaces an old one, delete the old one. Update every import that referenced it. Search the codebase for any remaining references. Orphaned code is one of the most common review findings and it's entirely preventable.

**Naming: explicit over abbreviated.** `async_call_llm` not `acall_llm`. `container_pool` not `pool`. Read existing names before choosing new ones.

Work in a red/green/refactor loop:
1. Run the focused failing test or executable check
2. Implement the smallest coherent production change that should make it pass
3. Run the focused test or check again
4. Refactor only after green, while keeping the test green
5. Repeat until the planned behavior is covered

During implementation:
- Do not broaden the public interface without updating the test first
- Do not test private implementation details unless the repo already does that for this kind of code
- Add regression tests when you discover missing edge cases
- Keep tests focused on behavior, not mocks of your own implementation

## Phase 4: Verify

After implementation, verify that what you wrote actually works. This phase catches the things that "looks right on a read-through" misses — integration bugs, signature mismatches, orphaned references.

### 4a. Trace every import and call site

For each file you created or modified:
- Confirm every import resolves to a real module and a real name in that module
- Confirm every function/method call matches the actual signature (argument count, names, types)
- Confirm async functions are awaited
- Confirm you're not treating Optional returns as non-Optional

This is different from Phase 1 — in Phase 1 you read APIs you *planned* to use. Here you're verifying what you *actually* wrote, which may have drifted during implementation.

### 4b. Run the code

Run the test suite. If the project has a way to exercise the new code path (even partially), do that too. A passing review means nothing if the code throws an ImportError on first execution.

If tests fail, fix them before presenting. If you can't fix a test failure, flag it explicitly — don't silently skip it.

### 4c. Dead code sweep

Search the entire codebase for references to anything you replaced or deleted:
- Old module names in import statements
- Old function names in call sites
- Old file paths in configs or documentation

If you find orphaned references, fix them now.

### 4d. Slop sweep

Review every file you created or modified for AI-generated patterns that don't match the surrounding code:

- Comments a human wouldn't write, or that are inconsistent with the file's comment style
- Defensive checks or try/catch blocks that are abnormal for that area (especially on trusted/validated codepaths)
- Redundant intermediate variables (`const result = foo(); return result;`) and unnecessary else after return/throw
- Duplicated logic that should be a loop or shared helper
- Verbose tests — duplicate setup that belongs in beforeEach/setUp, redundant assertions testing the same thing
- New files, exports, options, parameters, or helpers with no caller in this diff or in the existing repo (code can drift speculative during implementation even when the plan was tight)

The bar: would this code look out of place if a teammate who wrote the rest of the file reviewed it? If yes, fix it.

### 4e. TDD completion check

Confirm the tests still protect the intended interface:
- Every acceptance criterion is covered by a test or explicit verification check
- New or changed tests were observed failing before implementation, unless updating existing tests was the correct path
- Tests exercise public behavior and intended usage ergonomics
- Targeted tests pass
- Relevant broader checks pass
- No skipped tests, TODOs, placeholders, or overfit assertions were introduced

### 4f. Completeness check

Walk through the plan point by point. Is every requirement addressed? Are there any TODOs, stubs, or placeholder implementations? If so, flag them — the user should know what's incomplete.

For inventory-driven work, also walk the ground-truth inventory point by point. Confirm every listed producer, subject, route, schema variant, payload shape, or call path is handled as planned, covered by focused tests where risk warrants it, or explicitly left out of scope.

### 4g. Post-implementation challenge pass

Review the completed diff as if an adversarial reviewer will inspect it next:
- Does any touched call site now pass the wrong arguments, miss an await, or rely on an import that does not exist?
- Does the implementation handle every in-scope item from the ground-truth inventory?
- Did you leave an old path, name, command, fixture, generated artifact, or config reference behind?
- Are tests proving the public behavior, or only the helper you happened to write?
- Would any likely `must-change` finding survive the burden of proof: exact broken step, violated repo fact, concrete failure, inside scope, smallest fix?

Fix proven blockers before final response. Downgrade or reject speculative concerns instead of growing the implementation.

## Phase 5: Present

Give a concise summary:

1. **What was built** — 2-3 sentences
2. **Files changed** — list with one-line descriptions
3. **Deviations from plan** — anything you changed and why (if none, say "None")
4. **TDD loop** — what test/check failed first, and what now passes
5. **Self-review findings** — anything you caught and fixed during Phase 4
6. **Open questions** — anything you're unsure about

No fluff. No test plan section. Don't narrate your process — just present the result.
