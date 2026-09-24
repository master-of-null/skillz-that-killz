# Personal Engineering Preferences

## Communication
- Be succinct and direct. Always be explicit, don't use fancy prose, or words that muddy definitions. I need clarity and simple prose. 
- Prefer short answers unless additional detail is necessary.
- Do not explain obvious implementation details.
- When presenting alternatives, recommend one rather than enumerating every possibility.
- Arguably the most important rule: Always think to yourself "Stop using jargon and speak coherently. State it more simply and concisely, like one human talking to another.""

## Engineering Philosophy
- Prefer pragmatic, boring solutions over clever ones.
- Apply YAGNI to code and reviews. Separate “technically possible” from “worth fixing for this product.” Recommend fixes only when concrete product impact justifies their cost and complexity. Omit speculative or negligible concerns.
- Prefer the smallest change that cleanly solves the current problem.
- Avoid unnecessary abstractions, layers, configuration, and indirection.
- Do not introduce a new dependency when the existing stack can reasonably solve the problem.
- Do not refactor unrelated code while implementing a change.
- Prefer explicit, readable code over overly generic or DRY code.
- Abstract only when there is meaningful repetition or a clear architectural boundary.

## Implementation
- Follow existing repository conventions unless there is a strong reason not to.
- Preserve backwards compatibility when practical, but do not add complexity solely for speculative compatibility.
- Prefer targeted tests for changed behavior over excessive test coverage.
- Point out meaningful risks, but do not over-engineer around low-probability edge cases.
