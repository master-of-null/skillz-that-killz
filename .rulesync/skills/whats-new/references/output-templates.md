# Output Templates & Formatting

Reference file for `/whats-new` — load when generating output for any phase.

## Help Mode / Quickstart Guide

Show this when the user types "help" at any point:

```
╔════════════════════════════════════════════════════════════════════════════╗
║                          /whats-new QUICKSTART                             ║
╚════════════════════════════════════════════════════════════════════════════╝

  THE BASICS
  ──────────
  /whats-new reads the Claude Code changelog, scans your skills/agents/
  hooks, and shows a dashboard of improvements you haven't adopted yet.
  It remembers where you left off — next run only reviews new versions.

  AT THE DASHBOARD
  ────────────────
    "3"           → full detail view for item #3
    "1-4"         → walk through items 1, 2, 3, 4
    "batch"       → implement all Quick Wins at once
    "implement 2" → implement item #2 right now
    "tickets 5-9" → create tickets for items 5-9
    "done #3"     → mark item #3 as done (no external tool needed)
    "rescan"      → refresh after manual changes

  INSIDE A DRILL-DOWN
  ───────────────────
    "implement"   → make the change now
    "next"/"prev" → navigate items
    "skip"        → skip this one
    "back"        → return to dashboard

  SCOPING
  ───────
  /whats-new                    → global + current directory
  /whats-new for my-repo        → target a specific repo
  /whats-new last 3 versions    → only recent changes
  /whats-new just hooks         → filter to one area
  /whats-new --compare a b      → compare multiple repos

  CHECKPOINT MANAGEMENT
  ─────────────────────
  The checkpoint tracks what you've acted on, not just what you've read.
    "show checkpoint"              → see current checkpoint
    "reset checkpoint to 2.1.70"  → start reviews from 2.1.70
    "reset checkpoint back 3"     → go back 3 versions
    "clear checkpoint"            → next run reviews everything

  DRY RUN
  ───────
  Add --dry-run to preview everything without writing any files:
    /whats-new --dry-run            → full preview, no changes
    /whats-new --dry-run batch      → preview all Quick Wins
  Implementations show "WOULD WRITE:" diffs. Checkpoint is never saved.

  ITEM MANAGEMENT (no external tool needed)
  ──────────────────────────────────────────
    "done #3"       → mark as done
    "skip #5"       → skip with reason
    "note #7 ..."   → add a note
    "promote #10"   → move to a higher tier

  WHEN YOU'RE DONE
  ────────────────
  Say "done", "exit", or "quit". You'll be asked whether to advance
  the checkpoint. Browsing alone doesn't move it.

  TIPS
  ────
  • Everything is natural language — "do the quick wins" works
  • Items keep their numbers all session — #3 is always #3
  • "rescan" after implementing to see updated adoption %
  • Use --dry-run to explore safely before committing to changes
  • /whats-new works across sessions — checkpoint persists on disk
```

After showing help, return to wherever the user was (dashboard or drill-down).

---

## Formatting Rules

- Use standard markdown tables (`| col |` with `|---|` separator)
- No box-drawing characters (`+---+`, `╔═╗`) inside table cells or for table borders
- Box-drawing is ONLY for section banners and decorative headers
- Keep the changelog summary scannable — one line per feature, no paragraphs
- Group by theme (breaking, highlights, previews, fixes), not chronologically
- Order highlights by importance/impact, not by version number
- Cap highlights at ~15 most important entries
- For fixes, only show ones relevant to the user's stack or commonly encountered
- **If `just [area]` is active:** filter the changelog summary to ONLY show entries relevant
  to that area. State the filter in the banner: `Reviewing [N] versions — filtered to: hooks`

## Phase 0: Banner

**Important: use ONLY ASCII characters between the `║` borders** — Unicode chars
like `★`, `→`, `—`, `•` have ambiguous display widths and cause misalignment.
Use `*`, `->`, `--`, `/` instead. Count characters carefully: each content line
must be exactly 74 chars between the outer `║` pair (76 total with borders).

### Normal mode
```
╔════════════════════════════════════════════════════════════════════════════╗
║  Claude Code v2.1.81  /whats-new                                           ║
║  Reviewing 245 versions (v0.2.21 -> v2.1.81)                               ║
║  First run -- reviewing everything                                         ║
╠════════════════════════════════════════════════════════════════════════════╣
║  Tip: "last 3 versions" / "just hooks" / "from 2.1.70" / "--dry-run"       ║
╚════════════════════════════════════════════════════════════════════════════╝
```

### Dry-run mode
```
╔════════════════════════════════════════════════════════════════════════════╗
║  Claude Code v2.1.81  /whats-new  [DRY RUN]                                ║
║  Reviewing 245 versions (v0.2.21 -> v2.1.81)                               ║
║  First run -- no files will be written                                     ║
╠════════════════════════════════════════════════════════════════════════════╣
║  Tip: "last 3 versions" / "just hooks" / "from 2.1.70"                     ║
╚════════════════════════════════════════════════════════════════════════════╝
```

## Phase 0: Changelog Summary Example

```
## What shipped

### Breaking / Deprecations

| Version | Change                                                    |
|---------|-----------------------------------------------------------|
| v2.1.68 | Opus 4.0 and 4.1 removed — auto-migrated to Opus 4.6     |

### Highlights (new capabilities)

| Version | Feature                        | What it enables                        |
|---------|--------------------------------|----------------------------------------|
| v2.1.80 | `effort` skill frontmatter     | Override thinking depth per skill      |
| v2.1.78 | `StopFailure` hook             | React to rate limits / API errors      |
| v2.1.76 | `PostCompact` hook             | Run logic after context compaction     |
| v2.1.0  | `context: fork` for skills     | Run skills without bloating context    |
| v1.0.57 | `model` in skill frontmatter   | Override model per skill (cost opt)    |

### Research previews (behind flags)

| Feature        | Flag / Status                              | Watch for               |
|----------------|--------------------------------------------|-------------------------|
| Agent Teams    | `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`   | Multi-agent collab      |

### Notable fixes & improvements

- Worktree sessions now load skills/hooks correctly (v2.1.78)
- 1M context for Opus 4.6 on Max/Team/Enterprise (v2.1.75)
```

## Phase 0: Menu

```
What next?

  [analyze]       Scan my stack and show what applies to me
  [N]             Tell me more about a feature from the list above
  [just X]        Focus on one area (hooks, skills, agents, etc.)
  [for repo-name] Analyze a specific repo instead of global
  [exit]          I've seen enough — save checkpoint and exit

  Tip: Say "just hooks" or "just agents" to skip to a specific area.
       Say "for my-project" to target a specific repo's stack.
```

## Phase 1: Stack + Adoption Tables

```
## Your stack

| Component   | Count | Scope   | Names                                     |
|-------------|-------|---------|-------------------------------------------|
| Skills      |    18 | global  | deploy, review, triage, ...               |
| Agents      |     3 | global  | code-reviewer, health-check, ...          |
| Hooks       |     6 | global  | PreToolUse(2), Stop(2), StopFailure, ...  |
| MCP Servers |     0 | —       | —                                         |
| Plugins     |     1 | global  | marketplace                               |

## Frontmatter adoption

| Feature         | Skills     | Agents    |
|-----------------|------------|-----------|
| model           | 15/18 83%  | 3/3 100%  |
| effort          | 14/18 78%  | —         |
| context: fork   | 4/18 22%   | —         |
| memory          | —          | 3/3 100%  |
| disallowedTools | —          | 1/3 33%   |
| hooks (scoped)  | 0/18 0%    | 0/3 0%    |
```

### Phase 1 Menu

Always show a context summary before the menu:

```
---
Stack: [N] skills, [N] agents, [N] hooks | Lowest adoption: [feature] at [N]%
---

What next?

  [dashboard]   Show the improvement dashboard
  [just X]      Focus on one area (hooks, skills, agents, etc.)
  [compare]     Compare adoption across multiple repos
  [back]        Go back to the changelog summary
  [exit]        Save checkpoint and exit

  Tip: Low adoption % = biggest opportunity. "just hooks" skips straight
       to hook recommendations. "compare" shows how other repos stack up.
```

## Phase 3: Dashboard Format

Use `## ` markdown headers for tier sections. Standard markdown tables only.

```markdown
## Improvement Dashboard

Reviewing v[start] → v[end] | CC v[VERSION] | Target: [scope]

---

### !! Breaking / Urgent ([count])

| #  | Issue                    | Affects        | Action   | Since   |
|----|--------------------------|----------------|----------|---------|
| 1  | Opus 4.0/4.1 removed     | model pins     | verify   | v2.1.68 |

### ** Must Have ([count])

| #  | Feature                   | Affects          | Effort   | Savings  | Since   |
|----|---------------------------|------------------|----------|----------|---------|
| 2  | `effort` on [N] skills    | [N] skills       | ~5 min   | ~40% tk  | v2.1.68 |
| 3  | `model` on [N] skills     | [N] skills       | ~5 min   | ~60x $   | v1.0.57 |

### ++ Useful ([count])

| #  | Feature                    | Affects               | Effort   | Since   |
|----|----------------------------|-----------------------|----------|---------|
| 5  | `hooks` in skill FM        | implement, complete   | ~30 min  | v2.1.0  |
| 6  | `once: true` for hooks     | SessionStart hooks    | ~10 min  | v2.1.0  |

### -- Nice to Have ([count])

| #  | Feature                    | Affects              | Effort   | Since   |
|----|----------------------------|----------------------|----------|---------|
| 10 | `PostCompact` hook         | long sessions        | ~20 min  | v2.1.76 |
| 11 | `ConfigChange` hook        | security auditing    | ~20 min  | v2.1.49 |

### >> Quick Wins (batch in one commit)

| #  | Change                 | File                  | ~Time  |
|----|------------------------|-----------------------|--------|
| 14 | Add `effort: high`     | whats-new/SKILL.md    | 1 min  |
| 15 | Add `effort: low`      | summary/SKILL.md      | 1 min  |

### Not applicable

| Feature                  | Why                                    |
|--------------------------|----------------------------------------|
| `--channels` push        | No channel MCP servers configured      |
| Voice mode               | Not using voice mode                   |

### Done (previously implemented)

| Feature                  | When         | Status                  |
|--------------------------|--------------|-------------------------|
| (none — first run)       | —            | —                       |

---

**[N] actionable | [N] done | Est: ~[time] | Quick wins: ~[time]**
```

### Dashboard Menu

**Always re-show the summary line before the menu** so the user has context
without scrolling. After every action (drill-down, implement, skip, etc.),
re-display this block before prompting:

```
---
[N] actionable | [N] done | [N] skipped | Quick wins: #14-#16
Items: !! #1 | ** #2-#4 | ++ #5-#9 | -- #10-#13 | >> #14-#16
---

What would you like to do?

  [N]        Drill into item #N (e.g., "3", "tell me about #8")
  [all]      Walk through all items one by one
  [select]   Pick items to act on (e.g., "select 1-4,14-16")
  [batch]    Implement all Quick Wins right now
  [implement N,N,N]  Implement specific items
  [tickets]  Create tickets for selected items
  [plan]     Save improvement plan to memory
  [export]   Write plan to a file
  [rescan]   Re-read stack and refresh dashboard (after manual changes)
  [exit]     Save checkpoint and exit

  Or type naturally: "implement the must-haves", "skip hooks",
  "do quick wins then #2 and #3", "create tickets for useful tier"

  Tip: Start with [batch] to grab all Quick Wins in one shot (~3 min).
       Use [select 1-4] to cherry-pick a range, or [tickets] to defer
       items to your issue tracker without implementing now.
```

## Phase 4: Drill-Down Format

```
━━━ #[N]: [Title] ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  Tier:     [tier]  |  Version: v[X.Y.Z]+  |  Effort: ~[N] min  |  Scope: [global/project]
  Affects:  [skill-a, skill-b, ...]

**What it does**
[1-3 sentence explanation of the feature]

**Why it matters here**
[Why specifically for THIS user's stack. Name skills/agents affected.]

  Adoption: [N]/[total] skills ([names that have it])
  Gap:      [N] skills should add it

**Before → after**
[Side-by-side frontmatter diff showing the change]

**Cost impact**
[Order-of-magnitude estimate or qualitative impact]

**Synergies**
[Which other items combine well with this one]

**Rollback**
[How to undo if it causes issues]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Viewing #[N] of [total] | Tier: [tier] | [N] done, [N] remaining

  [implement]   Implement this now
  [next]        Next item (#[N+1])
  [prev]        Previous item (#[N-1])
  [skip]        Skip, go to next
  [back]        Return to dashboard
  [tickets]     Create a ticket for this

  Tip: "implement" makes the change now. "tickets" defers to your issue tracker.
       "next"/"prev" browses without committing to anything.
```

## Phase 5: Implementation Outputs

### Single item verification
```
Implemented #[N]: [description]

| File                              | Status    |
|-----------------------------------|-----------|
| ~/.claude/skills/[name]/SKILL.md  | verified  |

[N]/[N] files verified.

---
Implemented: #[N] | Remaining: [N] items | Quick wins left: [N]
---

  [next]     Next item (#[N+1]: [title])
  [back]     Return to dashboard
  [rescan]   Refresh adoption data
  [done]     Save checkpoint and exit
```

### Batch progress
```
  [x] #2  effort on 14 skills — 14/14 verified
  [>] #3  model on 12 skills — implementing...
  [ ] #4  context: fork on 5 skills
```

### Implementation summary
```
━━━ IMPLEMENTATION SUMMARY ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| #  | Item                        | Files | Verified | Scope  |
|----|-----------------------------|-------|----------|--------|

Total: [N] items, [N] files changed, [N]/[N] verified

---
Session: [N] implemented, [N] remaining | Adoption: model [N]%, effort [N]%, fork [N]%
---

  [commit]    Commit all changes
  [tickets]   Create tickets for remaining items
  [rescan]    Refresh dashboard with updated adoption data
  [back]      Return to dashboard
  [done]      Save checkpoint and exit

  Tip: Run [rescan] to see updated adoption %. [tickets] defers items
       to your issue tracker. Checkpoint only advances on [done].
```

### Dry-run single item
```
[DRY RUN] #[N]: [description]

  WOULD WRITE to ~/.claude/skills/[name]/SKILL.md:
  + [frontmatter line]          (after line [N]: "[context]")

[N] files previewed. No changes written.

  [next]     Next item
  [back]     Return to dashboard
  [done]     Exit dry run

  Tip: Run without --dry-run to apply these changes for real.
```

### Dry-run batch (use `[~]` instead of `[x]`)
```
  [~] #2  effort on 14 skills — 14 files previewed
  [~] #3  model on 12 skills — 12 files previewed
```

## Phase 8: Checkpoint Template

```markdown
---
name: What's new checkpoint
description: Last Claude Code version reviewed — diff only new releases next time
type: reference
---

**Last reviewed version:** [latest] (reviewed [date])
**Claude Code version at review:** [CURRENT_VERSION]
**Target:** [broad/global or specific repo path]

**Versions reviewed:** [start] → [end] ([count] versions)

**Items identified ([count]):**
- [list with numbers and version refs]

**Adoption at review time:**
- effort:         [N]/[total] skills
- model:          [N]/[total] skills
- context: fork:  [N]/[total] skills

**Actions taken:**
- Implemented: #N, #N ([count] items, [count] files, all verified)
- Tickets: #N → PROJ-1234, #N → PROJ-1235
- Skipped: #N, #N (reason)
- Remaining: #N, #N

**Status:** [summary]
```

## Phase 8: Exit Banner

```
╔════════════════════════════════════════════════════════════════════════════╗
║  Checkpoint saved at v[X.Y.Z]                                              ║
║  Next run reviews only new versions. Use --full to review everything.      ║
╠════════════════════════════════════════════════════════════════════════════╣
║  Reviewed:     [N] versions ([start] -> [end])                             ║
║  Found:        [N] actionable items                                        ║
║  Implemented:  [N] items ([N] files, all verified)                         ║
║  Tickets:      [N] created                                                 ║
║  Remaining:    [N] items for next time                                     ║
╚════════════════════════════════════════════════════════════════════════════╝
```
