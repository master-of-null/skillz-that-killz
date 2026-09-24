---
name: code-review
description: >-
  Exhaustive code review of the current branch's diff before pushing. Use this
  skill whenever the user asks to review their PR, review their branch, check
  changes before pushing, do a code review, pre-push review, audit their diff,
  or anything related to reviewing code changes on the current branch. Also
  trigger when the user says things like "what's wrong with my changes",
  "anything I'm missing before I push", "sanity check my branch", or "look over
  my PR". This skill is specifically for reviewing uncommitted or
  committed-but-not-pushed changes against a target branch — not for reviewing
  entire files or codebases from scratch.
---
# Pre-Push Review

An exhaustive single-pass code review that catches all bugs and logic errors in your branch diff before you push. The whole point of this skill is to eliminate the "fix one thing, push, CI finds another thing, repeat" cycle by surfacing everything in one shot.

## Philosophy

This is not a style linter or a nitpick machine. The review focuses on **bugs and logic errors** — things that would cause incorrect behavior, crashes, data corruption, or subtle wrongness at runtime. Think like a senior engineer doing a final review before approving a PR: what could actually break?

## Step 1: Gather Context

### Identify the diff

Determine the target branch and get the diff:

```bash
# Figure out the default/target branch
TARGET=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/origin/@@')
if [ -z "$TARGET" ]; then
  # Fallback: try main, then master
  TARGET=$(git branch -r | grep -E 'origin/(main|master)$' | head -1 | sed 's@.*origin/@@' | tr -d ' ')
fi

# Get the diff — committed changes on this branch vs target
git diff "$TARGET"...HEAD

# Also check for any uncommitted/staged changes
git diff
git diff --cached
```

Combine all three diffs into the full picture of what will land when this branch merges.

If the combined diff is very large (more than ~3000 lines), break the review into logical chunks by file or directory rather than trying to review everything in one shot. Process each chunk, then compile findings at the end.

### Read project conventions

Before reviewing, scan the repo root and common locations for conventions files. Read any that exist — they inform what the team considers correct:

```
.cursorrules
CONVENTIONS.md
CLAUDE.md
.github/CONTRIBUTING.md
docs/adr/          (read titles/recent ADRs, not all of them)
.eslintrc*         (just note the config exists and key rules)
tsconfig.json      (note strict mode, path aliases, etc.)
```

Don't spend more than a minute on this. The goal is to absorb enough context to avoid flagging things that are intentional project patterns. If no conventions files exist, that's fine — proceed with general best practices.

## Step 2: Review the Diff

Go through the diff systematically. For each changed file, examine:

1. **Logic errors** — incorrect conditionals, wrong operator, off-by-one, inverted boolean, missing early return, unreachable code
2. **Null/undefined hazards** — accessing properties on potentially null values, missing optional chaining, unsafe destructuring
3. **Async mistakes** — missing await, unhandled promise rejections, race conditions, fire-and-forget calls that should be awaited
4. **State bugs** — stale closures, missing dependency array entries in useEffect/useMemo, mutation of supposedly immutable data
5. **Data flow issues** — wrong variable used (copy-paste errors), function called with wrong arguments, return value ignored when it shouldn't be
6. **Edge cases** — empty arrays/objects not handled, division by zero, integer overflow, missing default case in switch
7. **Error handling gaps** — try/catch that swallows errors silently, missing error propagation, catch blocks that don't handle the error type they're catching
8. **AI slop** — comments a human wouldn't write or that are inconsistent with the file's style, defensive checks abnormal for the codebase (especially on trusted codepaths), redundant intermediate variables (`const result = foo(); return result;`), unnecessary else after return/throw, duplicated logic that should be consolidated, verbose tests with duplicate setup or redundant assertions

Focus on what the diff **changes or introduces**. Don't review unchanged code unless a change creates a new interaction with it (e.g., a function signature changed but a caller in the diff wasn't updated).

### What NOT to flag

- Style preferences (naming conventions, formatting) — unless it directly causes a bug
- Missing tests — that's a separate concern
- Performance optimizations — unless the code is clearly O(n²) on large datasets or has an obvious memory leak
- TODOs or incomplete features — unless they'll cause runtime errors
- Import ordering

## Step 3: Present Findings

Present all findings in a single structured report. Group by severity:

### Output Format

```
## 🔴 Critical (will cause bugs or crashes)

### [filename]:[line range] — [short description]
**What's wrong:** [Clear explanation of the bug]
**What happens:** [The concrete failure scenario — what a user would see or what data gets corrupted]
**Suggested fix:** [Brief description of how to fix it, not a full code block unless it's a one-liner]

---

## 🟡 Likely Issues (probably a bug, worth verifying)

### [filename]:[line range] — [short description]
**What's wrong:** [Explanation]
**Why it matters:** [What could go wrong]
**Suggested fix:** [How to address it]

---

## 🟢 Looks Good

[One or two sentences on what the PR does well or a general thumbs-up if everything is clean.]
```

If there are zero findings, say so clearly: "No bugs or logic errors found in this diff. Looks good to push." Don't manufacture findings to seem thorough.

### Severity guide

- **🔴 Critical**: Will definitely or very likely cause incorrect behavior. Nulls that will throw, logic that produces wrong results, async bugs that lose data.
- **🟡 Likely Issue**: Could cause problems under specific conditions. Edge cases that aren't handled, error paths that silently fail, assumptions that might not hold.

Keep the report scannable. Engineers are busy. Each finding should be understandable in 10 seconds.

## Important Reminders

- Be exhaustive. The user is counting on you to find EVERYTHING in one pass — bugs, logic errors, and slop — so they don't have to push multiple times. If you're uncertain whether something is a real issue, include it as a 🟡 with your reasoning rather than omitting it.
- Respect project conventions. If the project uses a pattern that looks unusual but is documented in their conventions files, don't flag it.
- Stay focused on the diff. The user wants to know if their changes are safe to push, not a full codebase audit.
