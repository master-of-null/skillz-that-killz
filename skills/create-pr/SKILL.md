---
name: create-pr
description: >-
  Create or update a pull request when the user asks to open, publish, or revise
  one. Reconstruct final intent from the change and relevant Codex sessions,
  then produce concise reviewer-first PR copy; not for reviewing pull requests.
---
# Create PR

Publish the current change with the shortest truthful explanation that lets a reviewer understand its intent, scope, and verification.

## Establish the change

1. Read repository instructions and any PR template. Determine the base branch from repository or remote evidence; do not assume `main`.
2. Inspect commits and the complete diff from the merge base, plus staged and unstaged changes. Keep unrelated work out of the PR.
3. Reconstruct the user's final intent from the current conversation and available issue context. When local Codex history is available, inspect only recent interactive root sessions scoped to this repository and branch; use the issue key, changed paths, and timeframe to confirm relevance. Read only user-authored turns and summarize final decisions, constraints, and meaningful pivots; later decisions supersede abandoned ideas. Do not search unrelated sessions or copy transcript text into the PR; summarize relevant intent in your own words. Skip history when access or relevance is uncertain—it must not block PR creation. Turn evolution into rationale, not a chat timeline.
4. Reconcile the narrative with the code. The diff and actual check results are authoritative. Ask before publishing only when scope is ambiguous or intent and implementation materially disagree.

## Name the branch and PR

- Preserve an existing non-base branch, especially a Linear-generated branch, unless the user asks to rename it.
- When creating a branch, follow documented repository naming first. Otherwise use `<type>/<short-kebab-summary>` with the primary-intent type `feature`, `fix`, `chore`, `refactor`, `test`, `docs`, `ci`, `build`, `perf`, or `revert`.
- Use a Linear-provided branch name verbatim. If only an issue key is available, retain it after the type, for example `feature/abc-123-short-summary`.
- When currently on the resolved base branch, create and switch to the correctly named branch before committing.
- Follow the repository's title convention. Otherwise use a concise Conventional Commit title such as `feat: add retry limits`; map branch `feature` to title `feat`.

## Write the body

Preserve required template structure and hidden markers. Keep every field brief and factual. Without a template, use:

```markdown
## Summary
- <why the change exists and its intended outcome>
- <implementation choice or constraint a reviewer needs>

## Validation
- `<command>` — <result>
```

Use one to three summary bullets. Add a root cause for a fix or a reviewer note only when it materially helps review. In the fallback body, omit file inventories, chronological process, generic claims, and empty optional sections; preserve every required template section and marker. State checks that were not run and why.

## Publish

Perform only the mutations the user requested.

- To create or publish a PR, run the smallest relevant checks, stage and commit only the intended changes, push the branch, and create one PR against the resolved base. Update an existing PR instead of creating a duplicate.
- To revise an existing PR, change only the requested PR metadata unless the user also asked to publish code changes.
- When asked only for draft copy, return the title and body without changing git or GitHub state.

Use draft status only when requested or required by repository convention.

Return the PR link and any validation not run.
