---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work
---

# Finishing a Development Branch

## Overview

**Core principle:** Verify tests → Detect environment → Apply existing authorization → Integrate if requested → Clean up only task-owned worktrees.

**Announce at start:** "I'm using the finishing-a-development-branch skill to complete this work."

## Step 1: Verify Tests

Run the checks required by the repository and appropriate to the change,
including the full suite when required. An existing successful run on the
exact unchanged tree is valid evidence; repeat when changes or failures make
it stale.

**If relevant tests fail**, fix failures within scope before integration.
Report unresolved failures and do not claim the work is ready:

```
Tests failing (<N> failures). Must fix before completing:

[Show failures]
```

**If tests pass:** continue to Step 2.

## Step 2: Detect Environment

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
# Capture now, while still inside the workspace — Step 5 changes directory
# before cleanup (Step 6) needs this value
WORKTREE_PATH=$(git rev-parse --show-toplevel)
```

Also run `git rev-parse --show-superproject-working-tree`; a submodule
can have different Git directories without being a disposable worktree.
Inspect `git worktree list --porcelain` to identify actual checkout paths.

| State | Integration | Cleanup |
|-------|-------------|---------|
| Normal repo or submodule checkout | Follow user request | No worktree cleanup |
| Linked worktree, named branch | Follow user request | Provenance-based (Step 6) |
| Linked worktree, detached HEAD | Create a branch if needed for the authorized action | Provenance-based (Step 6) |

Detached HEAD does not imply a sandbox restriction or prove the app owns
the workspace. Follow actual permissions and recorded creation provenance.

## Step 3: Determine Base Branch

The base branch is whatever this work forked from — usually named in the
plan, the conversation, or the branch's upstream. If it is not already
known, inspect the repository and remote branch configuration. Ask only if
a requested merge or PR still has an ambiguous target. Do not ask about a
base branch when simply leaving the completed work in place.

## Step 4: Apply the User's Integration Choice

If the user already requested a merge, push, or PR, complete that authorized
action without another permission menu. If they only asked for implementation,
leave the changes in place and report the result. When they ask you to choose
how to integrate but have not specified enough, recommend one option and ask
only for the missing decision:

1. Merge back to the verified base branch locally
2. Push and create a pull request
3. Keep the work as-is

For detached HEAD, create a named branch when the authorized integration needs
one and permissions allow it. Do not discard work unless explicitly requested.

## Step 5: Execute Choice

### Option 1: Merge Locally

```bash
# Identify the base checkout with git worktree list --porcelain first.
# Preserve unrelated local changes; do not switch a checkout another task uses.
cd "<verified-base-checkout>"

# Merge first — verify success before removing anything
git checkout <base-branch>
# Update the base only when needed by the requested integration.
git merge <feature-branch>

# Verify tests on merged result
<test command>
```

If tests fail on the merged result: stop, leave the worktree and branch in
place, and investigate — nothing has been pushed, so the merge is local
and recoverable.

Once the merged result is green, apply Step 6. Delete a merged feature branch
only when this task created it or the user requested branch cleanup, and no
preserved worktree still uses it:

```bash
git branch -d <feature-branch>
```

### Option 2: Push and Create PR

```bash
git push -u origin <feature-branch>
# From a detached HEAD, name the new branch on the remote:
# git push origin HEAD:refs/heads/<new-branch>
```

Then use create-pr when available to create the pull/merge request against
<base-branch> with the forge's tooling — its CLI if one is available, or the creation URL most forges
print when you push — following the repo's PR template and conventions if
present, and report the URL to your human partner.

Keep the worktree — your human partner iterates on PR feedback there.

### Option 3: Keep As-Is

Report: "Keeping branch <name>. Worktree preserved at <path>."

### If your human partner asks to discard the work

This path exists only for an explicit request to discard the work. Inspect the
exact branch, commits, and uncommitted files first. If that scope was already
clearly authorized, carry it out without requiring a special confirmation word.
If the request is ambiguous or inspection finds additional work outside that
scope, list what would be lost and ask before deleting it.

Move outside the worktree to a verified checkout, then use Step 6 for the
authorized cleanup. Delete the feature branch only when the user's discard
request covers it; preserve any unrelated commits or files.

## Step 6: Cleanup Workspace

**Runs after a successful local merge or an authorized discard.** Options 2 and 3
preserve the worktree. Both callers have already changed directory to the
verified checkout — worktree removal must run from outside the worktree —
and use the `GIT_DIR`/`GIT_COMMON`/`WORKTREE_PATH` values captured in
Step 2, from before that directory change.

**For a normal repo or submodule checkout:** No worktree cleanup. Done.

**Only if the session records show this task created `WORKTREE_PATH` and the
user's request permits cleanup:** remove that specific worktree. Directory
names such as `.worktrees/` do not prove ownership. Existing or app-managed
worktrees stay in place unless the user specifically requests their removal.

```bash
git worktree remove "$WORKTREE_PATH"
```

**If removal is refused** (`contains modified or untracked files`), inspect
`git -C "$WORKTREE_PATH" status --porcelain -uall` and any ignored artifacts.
Preserve files outside the cleanup authorization. If the user already asked
to discard these exact files, follow that instruction; otherwise explain what
would be lost and ask before deleting them. Never use `--force` merely to get
past a refusal.

**If ownership is unknown:** leave the workspace in place and report its path.
Do not clean other worktrees or prune registrations as unrelated housekeeping.

## Quick Reference

| Option | Merge | Push | Keep Worktree | Cleanup Branch |
|--------|-------|------|---------------|----------------|
| 1. Merge locally | yes | - | If not task-owned | If task-owned or requested |
| 2. Create PR | - | yes | yes | - |
| 3. Keep as-is | - | - | yes | - |
| Discard (explicit request only) | - | - | Only remove if authorized | Only if authorized |

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Tests passed before the last edit" | Verify the exact tree being integrated; evidence from an unchanged tree can be reused. |
| "They obviously want it merged" | Follow their actual request. Prior merge authorization counts; implementation alone does not imply a merge. |
| "They seem done with this feature — I'll discard it" | Discard only work explicitly covered by the user's request. |
| "The PR is up, so the worktree is clutter now" | PR feedback gets fixed in that worktree. It stays until the work lands. |
| "This other worktree looks stale — I'll clean it too" | Clean up only a worktree this task demonstrably created, unless the user explicitly authorized a wider cleanup. |
| "Removal refused — `--force` is just finishing the cleanup" | Inspect the files and preserve anything outside the user's explicit deletion authorization. |
| "The merged-result failure is probably flaky" | A failing merged result stops everything. Branch and worktree stay put while you investigate. |
| "The base branch is obviously main" | Verify the target from the request and repository, then ask only if it remains ambiguous. Merging into the wrong base is expensive to undo. |
| "The push was rejected — force-push will fix it" | A rejected push means the remote moved. Investigate; force-push only on your human partner's explicit request. |
