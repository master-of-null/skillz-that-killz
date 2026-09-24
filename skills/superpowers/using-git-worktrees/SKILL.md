---
name: using-git-worktrees
description: Use when feature work needs isolation from the current checkout, or when checking an existing Codex worktree before an implementation plan
---

# Using Git Worktrees

## Overview

Use an isolated workspace when the task needs it. Small scoped changes can
stay in the current checkout, and an explicit user preference to work there
takes precedence. Prefer a native tool that supports the requested operation;
otherwise use Git.

**Core principle:** Detect existing isolation first. Then use native tools. Then fall back to git. Never fight the harness.

**Announce at start:** "I'm using the using-git-worktrees skill to set up an isolated workspace."

## Step 0: Detect Existing Isolation

**Before creating anything, check if you are already in an isolated workspace.**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
BRANCH=$(git branch --show-current)
```

**Submodule guard:** `GIT_DIR != GIT_COMMON` is also true inside git submodules. Before concluding "already in a worktree," verify you are not in a submodule:

```bash
# If this returns a path, you're in a submodule, not a worktree — treat as normal repo
git rev-parse --show-superproject-working-tree 2>/dev/null
```

**If `GIT_DIR != GIT_COMMON` (and not a submodule):** You are already in a linked worktree. Skip to Step 2 (Project Setup). Do NOT create another worktree.

Report with branch state:
- On a branch: "Already in isolated workspace at `<path>` on branch `<name>`."
- Detached HEAD: "Already in isolated workspace at `<path>` (detached HEAD)."
  Detached HEAD does not establish ownership or prohibit branch creation.
  Check the actual permissions and task instructions before changing Git state.

**If `GIT_DIR == GIT_COMMON` (or in a submodule):** You are in a normal repo checkout.

Honor the user's existing workspace preference. Create a worktree when isolation
helps protect concurrent or unrelated work; it is a reversible setup step and
does not require a separate permission question when already within scope.
If isolation is unnecessary, work in place and skip to Step 2.

## Step 1: Create Isolated Workspace

**You have two mechanisms. Try them in this order.**

### 1a. Native Worktree Tools (preferred)

Inspect the live tools and their restrictions. Use a native tool only if it
supports this workspace operation within the user's request. Codex app task
creation and handoff tools are not general-purpose worktree commands: do not
create a user-owned task for internal isolation or try to hand off the calling
task when its schema forbids it. An existing app-managed worktree already
satisfies isolation.

If no suitable native tool applies, use Step 1b. Record the exact worktree path
and how it was created so finishing-a-development-branch can determine whether
this task owns cleanup.

### 1b. Git Worktree Fallback

**Use this when Step 1a does not apply.** Create a worktree manually using Git.

#### Directory Selection

Follow this priority order. Explicit user preference always beats observed filesystem state.

1. **Check your instructions for a declared worktree directory preference.** If the user has already specified one, use it without asking.

2. **Check for an existing project-local worktree directory:**
   ```bash
   ls -d .worktrees 2>/dev/null     # Preferred (hidden)
   ls -d worktrees 2>/dev/null      # Alternative
   ```
   If found, use it. If both exist, `.worktrees` wins.

3. **If there is no other guidance available**, default to `.worktrees/` at the project root.

#### Safety Verification (project-local directories only)

**MUST verify directory is ignored before creating worktree:**

```bash
git check-ignore -q "$LOCATION"
```

**If NOT ignored:** Add the chosen directory to the repository-local exclude
file (`git rev-parse --git-path info/exclude`), or follow the repository's
existing `.gitignore` convention. Verify again before creating the worktree.
Do not create an unrelated commit just for this setup step.

**Why critical:** Prevents accidentally committing worktree contents to repository.

#### Create the Worktree

```bash
# Determine path based on chosen location
WORKTREE_PATH="$LOCATION/$BRANCH_NAME"

git worktree add "$WORKTREE_PATH" -b "$BRANCH_NAME"
cd "$WORKTREE_PATH"
```

**Permissions:** A denied `git worktree add` is a permission issue, not proof
that Git or worktrees are unsupported. Use the runtime approval mechanism if
isolation is required and the action is authorized. Work in place only when
that still satisfies the user's request and does not put unrelated work at
risk. Explain any remaining limitation; do not silently abandon requested
isolation. Record worktree ownership after successful creation.

## Step 2: Project Setup

Use the repository's documented setup and package manager. Reuse installed
dependencies when ready; install only when needed. Common examples:

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

## Step 3: Verify Clean Baseline

For code changes, establish the relevant baseline with the repository's
required checks and tests for the affected behavior. Reuse successful results
from the same unchanged tree when available. Read-only work or documentation
edits do not automatically require a test suite.

```bash
# Use project-appropriate command
npm test / cargo test / pytest / go test ./...
```

**If tests fail:** Investigate enough to distinguish existing failures from
task regressions. Record unrelated baseline failures and continue authorized
work when they do not block validation. Ask only if an unresolved failure
prevents a safe next step.

**If tests pass:** Report ready.

### Report

```
Worktree ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

## Quick Reference

| Situation | Action |
|-----------|--------|
| Already in linked worktree | Skip creation (Step 0) |
| In a submodule | Treat as normal repo (Step 0 guard) |
| Suitable native workspace tool available | Use within its schema and authorization (Step 1a) |
| No suitable native tool | Git worktree fallback (Step 1b) |
| `.worktrees/` exists | Use it (verify ignored) |
| `worktrees/` exists | Use it (verify ignored) |
| Both exist | Use `.worktrees/` |
| Neither exists | Check instruction file, then default `.worktrees/` |
| Directory not ignored | Exclude chosen directory, then verify |
| Permission error on create | Use runtime approval or a safe authorized alternative |
| Tests fail during baseline | Investigate, record pre-existing failures |
| No package.json/Cargo.toml | Skip dependency install |

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "I'm obviously not in a worktree — no need to check" | Run Step 0. Harness-created isolation and submodules both fool eyeballing; the detection commands settle it. |
| "A task-creation tool can substitute for any workspace tool" | User-owned task creation has its own authorization rules. Use the current schema; Git is valid when no suitable native workspace tool applies. |
| "The worktree directory is surely ignored already" | Run `git check-ignore`. An unignored worktree directory commits the whole tree into the repo. |
| "Any directory name works" | Explicit instructions beat an existing project-local directory, which beats the `.worktrees/` default. |
| "The workspace is fresh — baseline checks can wait" | Establish the baseline relevant to the change, reusing valid results for an unchanged tree. Investigate failures that prevent validation. |
