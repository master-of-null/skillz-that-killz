---
name: adversarial-plan-review
description: >-
  Adversarial but calibrated, constructive, read-only review of implementation
  plans, architecture plans, and proposed code changes against repo-grounded
  evidence. Use when the user asks Codex to contest, challenge, poke holes in,
  sanity-check, validate assumptions, or adversarially review a plan before
  implementation, especially when they want findings classified as must-change,
  should-change, record-only, or reject without forcing blockers.
---
# Adversarial Plan Review

Contest the plan against actual repo behavior and stated long-term architecture. Do not try to force changes. Decide whether the plan is correct, incomplete, over-scoped, or under-evidenced. A clean review is a valid outcome.

This skill is read-only by default. Do not edit files unless the user separately asks for implementation after the review.

## Workflow

1. Identify the plan's goal, acceptance criteria, and explicit scope.
2. Read relevant repo files, tests, specs, tickets, and architecture docs.
3. Validate assumptions against actual producers, consumers, payloads, call sites, config, schemas, and runtime paths.
4. Try to clear the plan: look for evidence that the plan is sufficient before looking for reasons to block it.
5. **YAGNI scope check.** For every file, abstraction, interface, option, config flag, or helper the plan proposes to add, ask what justifies it. If the stated justification is speculative — "we might need this later," "future flexibility," "in case we extend it," "for symmetry with X" — label it `should-change` (de-scope) unless the user explicitly asked for the abstraction. If the item has callers elsewhere in the plan, documents a real contract or boundary (a type, schema, or error class one impl uses now), or is normal test/fixture infrastructure, it earns its place — not a YAGNI finding. Robustness for behavior that exists is fine; scaffolding for behavior nobody has asked for is not.
6. Classify each concern with one severity label.
7. Recommend whether to proceed as planned, proceed with scoped changes, or re-plan.

If the user explicitly asks for a sub-agent, parallel review, or delegated challenge, spawn a reviewer sub-agent with these same rules. Otherwise, perform the review locally.

## Severity Labels

Use exactly these labels:

- `must-change`: the implementation would be wrong, fail acceptance criteria, violate a contract, or likely break existing behavior.
- `should-change`: improves correctness, maintainability, or test quality without expanding the original scope.
- `record-only`: valid long-term concern, but not required for the current ticket or plan.
- `reject`: not supported by repo evidence, too speculative, or conflicts with the requested scope.

Do not turn `record-only` concerns into implementation scope. Do not expand scope unless the finding is `must-change`, or a `should-change` that stays inside the original acceptance criteria.

## Must-Change Burden Of Proof

Before labeling anything `must-change`, answer all five questions:

- What exact plan step or omission causes the problem?
- What exact repo contract, acceptance criterion, runtime path, schema, or caller expectation does it violate?
- What concrete failure, regression, or user-visible wrong behavior would plausibly happen?
- Why is that failure inside the requested scope, not a future hardening concern?
- What is the smallest plan change that would prevent the failure?

If any answer is missing, do not use `must-change`. Downgrade to `should-change`, `record-only`, or `reject`.

Use `should-change` for in-scope improvements, missing tests, or incomplete plan details that improve confidence but do not prove the planned implementation would fail. Use `record-only` for real risks that depend on future scope, broader architecture cleanup, scale, optional hardening, or unrequested polish. Use `reject` for concerns based on naming guesses, preferred style, hypothetical edge cases, or assumptions that repo evidence does not support.

Do not classify a concern as `must-change` just because:
- The plan omits an implementation detail a competent implementer could fill in while coding.
- Another design would be cleaner, more general, or more future-proof.
- A test would be useful but the missing test does not expose likely broken behavior.
- The reviewer is uncertain. Read more; if uncertainty remains, say so and choose a lower severity.

## Evidence Standard

For every `must-change` and `should-change` finding:
- Cite concrete files, functions, tests, docs, tickets, or specs.
- Explain the contract or behavior that makes the finding relevant.
- State the smallest scope adjustment that addresses it.

For `record-only` findings, explain why the concern is real and why it is not required now.

For `reject` findings, explain which repo fact or scope boundary invalidates the concern.

When facts are uncertain, say so. Do not present speculation as a blocker.

Before finalizing, reread every `must-change` finding and demote it unless it still satisfies the full burden-of-proof checklist.

## Output

Return a concise report with these sections:

1. **Must-Change Findings**
2. **Should-Change Findings**
3. **Record-Only Concerns**
4. **Rejected Or Weak Concerns**
5. **Final Recommendation**

Use `None` for empty sections. Do not invent findings to avoid an empty `Must-Change Findings` section.

The final recommendation must be one of:
- `proceed as planned`
- `proceed with scoped changes`
- `re-plan`

Recommend `proceed as planned` when there are no proven blockers and any concerns are record-only or rejected. Recommend `proceed with scoped changes` when localized `must-change` or in-scope `should-change` items should be handled before or during implementation. Recommend `re-plan` only when the plan's core approach conflicts with the goal, repo contracts, or acceptance criteria.

End with assumptions, known gaps, and confidence.
