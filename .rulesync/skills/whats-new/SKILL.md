---
name: whats-new
description: >
  Cross-reference Claude Code changelog against the current repo's skills,
  agents, hooks, and developer tooling, then categorize improvements by impact.
  Conversational and interactive — presents a visual dashboard, lets the user
  drill into items, select what to upgrade, and implements changes one-at-a-time
  or in batch. Repo-agnostic; discovers the stack dynamically. Remembers the
  last checkpoint so repeat runs only review new versions. Loops until the user
  says done. Use when user says "what's new", "changelog improvements", "stack
  audit", or "/whats-new".
targets:
  - '*'
---
# /whats-new Skill

Cross-reference Claude Code changelog against your **actual** stack — skills, agents, hooks,
MCP servers, plugins — then surface contextual, actionable improvements ranked by impact.
Interactive loop that continues until the user exits.

> **Key principle:** Don't just list changelog entries. Show the user *why* each feature matters
> to *their specific setup* with concrete examples drawn from their existing code.

**Reference files** (load when needed for output formatting and advanced features):
- `${CLAUDE_SKILL_DIR}/references/output-templates.md` — banners, menus, dashboard/drill-down/implementation templates
- `${CLAUDE_SKILL_DIR}/references/advanced.md` — cross-repo comparison, pain-point detection, adoption history, composability

## Usage Modes

```
/whats-new                                # Analyze global + cwd stack (checkpoint-aware)
/whats-new --full                         # Ignore checkpoint, review all versions
/whats-new --since 2.1.80                # Review from a specific version onward
/whats-new last 3 versions               # Review only the most recent 3 version entries
/whats-new from 2.3.0 to 2.5.0           # Review a specific range
/whats-new just hooks                     # Focus on a specific area
/whats-new drill #3                       # Jump straight to item #3 detail view
/whats-new implement #1 #3 #7            # Implement specific items immediately
/whats-new ~/Developer/my-project        # Analyze a specific repo's stack
/whats-new for my-project                # Target a repo by name (resolved from cwd)
/whats-new --compare repo-a repo-b       # Cross-repo adoption comparison
/whats-new compare all repos             # Compare all repos in ~/Developer
/whats-new reset checkpoint to 2.1.70   # Set checkpoint to a specific version
/whats-new reset checkpoint back 3      # Move checkpoint back 3 versions
/whats-new clear checkpoint              # Remove checkpoint — next run reviews all
/whats-new show checkpoint               # Show current checkpoint status
/whats-new --dry-run                     # Preview all recommendations without writing any files
/whats-new --dry-run implement #1 #3    # Dry-run a specific implementation
/whats-new help                          # Show interactive quickstart guide
```

All arguments are parsed **conversationally** — natural language always works.

---

## Argument Parsing

Parse the user's input to determine **scope**, **target**, and **mode**.

### Scope (which versions to review)

- **No args** → use checkpoint if available, otherwise review everything
- **`--full`** / **`everything`** / **`all`** → ignore checkpoint, review all versions
- **`--since X.Y.Z`** / **`from X.Y.Z`** → review from version X.Y.Z onward
- **`last N versions`** / **`last N updates`** / **`past N versions`**
  → count the N most recent version headings in the changelog.
  Note: this scopes the **changelog summary** (Phase 0) only. The dashboard (Phase 3)
  still shows ALL unadopted features — adoption gaps exist regardless of when shipped.
- **`from X.Y.Z to A.B.C`** → review only versions in that range (inclusive)
- **`just [area]`** / **`only [area]`** → filter to: hooks, skills, agents, MCP, plugins, worktrees, memory
- **`drill #N`** / **`tell me about #N`** → run Phase 0–2 silently (no output), then
  jump straight to the drill-down for item #N. If #N doesn't exist, show available range.
- **`implement #N ...`** → run Phase 0–2 silently, then implement items immediately
- **`--dry-run`** / **`dry run`** / **`preview`** → run everything normally but never write files.
  Implementations show `WOULD WRITE:` diffs. Checkpoint is never saved. Safe to explore.

### Checkpoint management

- **`reset checkpoint to X.Y.Z`** → update checkpoint file immediately and confirm.
- **`reset checkpoint back N versions`** → count N `## X.Y.Z` headings back. Confirm before writing.
- **`clear checkpoint`** → delete the checkpoint file from the project memory directory
  and remove its entry from MEMORY.md. Next run reviews everything.
- **`show checkpoint`** → display the current checkpoint version + what it recorded.

**Key rule: browsing vs. advancing.**

The checkpoint represents "I have reviewed AND acted on everything up to this version."
It only advances when the user has implemented or explicitly confirmed — not from browsing.

1. **Browsing** (`/whats-new from 2.1.70 to 2.1.75`) does NOT move the checkpoint.
2. **Implementing** items → checkpoint advances on explicit exit (Phase 8).
3. **Explicit exit** → user is asked to confirm checkpoint save.
4. **Resetting** → updates immediately. Next run in any session uses it.
5. **Re-running** `/whats-new` in same session → reads checkpoint from disk (not memory).
6. **Dry-run** → checkpoint is NEVER saved.

### Target (which stack to analyze)

- **No path/name** → **broad/global mode**: `~/.claude/` + `$PWD/.claude/` if it exists
- **Explicit path** → that directory's `.claude/` + global `~/.claude/`
- **Repo name** (`for my-project`) → resolve via `$PWD/<name>`, `~/Developer/<name>`,
  `~/Projects/<name>`, `~/<name>`. If ambiguous, ask.

**If the target path doesn't exist** or has no `.claude/` directory:
> "Path doesn't exist (or has no .claude/ config). Did you mean one of these? [list nearby repos]"

### Input validation

- **`last 0 versions`** → "0 versions means nothing to review. Did you mean `last 1`?"
- **`last -3 versions`** → "That's not a valid count. Try `last 3 versions`."
- **`from 2.1.81 to 2.1.70`** (backwards) → "That range is backwards. Swap? [yes / cancel]"
- **`--since 99.99.99`** → "Version not found. Latest is v[X.Y.Z]. Try `--since [X.Y.Z-1]`."
- **`just everything`** → treat as `--full`

---

## First-Run Onboarding

When no checkpoint exists (first time), show a welcome banner before anything else:

```
╔════════════════════════════════════════════════════════════════════════════╗
║  Welcome to /whats-new!                                                    ║
╠════════════════════════════════════════════════════════════════════════════╣
║                                                                            ║
║  1. I'll scan your skills, agents, and hooks                               ║
║  2. Cross-reference against the Claude Code changelog                      ║
║  3. Show you a dashboard of improvements ranked by impact                  ║
║  4. You pick what to implement -- I'll do the rest                         ║
║                                                                            ║
║  Just follow the prompts. Type "help" anytime for more options.            ║
╚════════════════════════════════════════════════════════════════════════════╝
```

On subsequent runs (checkpoint exists), skip the welcome.

---

## Help Mode

When the user types `help` at any point, show the quickstart guide from
`${CLAUDE_SKILL_DIR}/references/output-templates.md` (the QUICKSTART banner section).
After showing help, return to wherever the user was.

The help guide covers: THE BASICS, AT THE DASHBOARD, INSIDE A DRILL-DOWN, SCOPING,
CHECKPOINT MANAGEMENT, DRY RUN, WHEN YOU'RE DONE, and TIPS.

---

## Phase 0: Version & Changelog

The first thing the user sees.

### Step 0a: Detect Version & Get Changelog

```bash
claude --version 2>/dev/null || echo "unknown"
```

Store as `CURRENT_VERSION`. If "unknown", warn and proceed with all features marked available.

Read the changelog from `~/.claude/cache/changelog.md`.

**If the changelog doesn't exist or is empty:**
> "No changelog found. Run `/release-notes` first to populate it, then try `/whats-new` again."

**Token management:** Do NOT read the entire changelog. Instead:
1. Read only version headings first (`grep '^## '`) to get the version list
2. Based on review window, read only the relevant version sections
3. Cap the changelog summary at ~15 highlights

### Step 0b: Check for "nothing new" state

If checkpoint is at `CURRENT_VERSION` and no explicit scope override:

Show "You're up to date" banner. If checkpoint has unimplemented items (check "Remaining"
field or "Status: NOT yet implemented"), surface them:

> "No new versions, but your checkpoint has [N] unfinished items from [date].
> [continue] Resume working on remaining items | [--full] Start fresh | [exit]"

Otherwise show: `[analyze] [--full] [reset] [exit]`

### Step 0c: Present the Changelog Summary

Read `${CLAUDE_SKILL_DIR}/references/output-templates.md` for banner and table formats.

Group by theme (breaking, highlights, previews, fixes). Order by importance, not version.
Present the Phase 0 menu after. Wait for user response.

---

## Phase 1: Stack Discovery

Only after the user has acknowledged the changelog (says "analyze" or similar).

### Step 1a: Determine Target

Broad/global mode (default) or targeted repo mode per argument parsing.

### Step 1b: Discover the Stack

Inventory **dynamically** — never hardcode paths or names.

```bash
# Skills + agents
ls ~/.claude/skills/ ~/.claude/agents/ 2>/dev/null
ls <target>/.claude/skills/ <target>/.claude/agents/ 2>/dev/null

# Hooks + MCP servers (from settings)
cat ~/.claude/settings.json 2>/dev/null   # parse hooks{} and mcpServers{}
cat <target>/.claude/settings.json 2>/dev/null

# Plugins
ls ~/.claude/plugins/ 2>/dev/null

# MCP (project-level)
cat <target>/.mcp.json 2>/dev/null
```

For **each skill and agent**, read its frontmatter (model, effort, context, hooks, memory,
tools, disallowedTools). Critical for the adoption check.

### Step 1c: Build Adoption Matrix

For each recommendable feature, for each skill/agent, check if the frontmatter is set.
This drives: excluding fully-adopted items, showing partial adoption, preventing re-recommendation.

**Critical: always verify live, never trust checkpoint claims.**
Even if a prior checkpoint says "#3 was implemented," re-read the actual frontmatter now.
Implementations can be reverted, files can be reset, branches can be discarded. The
adoption matrix is built from **current file state**, not from checkpoint memory.

If a previously-implemented item is now missing from the files, surface it:
> "Note: #3 (context: fork on 5 skills) was implemented in a prior session but
> 2 of 5 skills no longer have it. Re-adding to the dashboard."

### Step 1d: Present Stack + Adoption

Read `${CLAUDE_SKILL_DIR}/references/output-templates.md` for table formats.
Show stack summary + adoption matrix. Present Phase 1 menu.

---

## Phase 2: Analysis & Categorization

### Classify, cross-reference, estimate cost impact

For each changelog entry in scope:
1. **Relevance** — applies to this stack?
2. **Already adopted?** — check adoption matrix
3. **Available?** — version <= CURRENT_VERSION?
4. **Impact** — cost, reliability, DX
5. **Effort** — minutes, hours, or days?

**Skip:** fully adopted features, bug fixes for tools not in stack, IDE-specific.

Each recommendation MUST include: what changed (version), why it matters to THIS stack
(name specific skills/agents), before/after example, effort estimate, cost impact, scope.

### Assign Numbers and Categorize

Give every recommendation a **persistent number** (#1, #2...) consistent throughout the session.

| Tier | Label | Criteria |
|------|-------|----------|
| 0 | `!! Breaking / Urgent` | Deprecations, breaking changes, security |
| 1 | `** Must Have` | Pain points, significant cost savings |
| 2 | `++ Useful` | Meaningful workflow improvements |
| 3 | `-- Nice to Have` | Small QoL, can defer |
| 4 | `>> Quick Wins` | Trivial (< 5 min), one-line changes |

Also maintain: **Not Applicable** (dismissed with reasons), **Needs Upgrade** (requires newer CC).

---

## Phase 3: Dashboard

Present ALL tiers in a single dashboard. Read `${CLAUDE_SKILL_DIR}/references/output-templates.md`
for the full dashboard format. Present the interactive menu. Wait for input. **Loop** after every action.

---

## Phase 4: Drill-Down

When user selects an item, show the drill-down view with: What it does, Why it matters here,
Before → after, Cost impact, Synergies, Rollback. Present navigation menu.
Read `${CLAUDE_SKILL_DIR}/references/output-templates.md` for format.

---

## Phase 5: Implementation

### Pre-implementation verification

Before implementing ANY item, look up the relevant section of the official
Claude Code documentation to confirm correct syntax. Search for the feature
name in the official Claude Code docs. Never implement from memory alone —
the docs are the source of truth for field names, allowed values, and
configuration format.

| Change type | Doc topic to search |
|-------------|---------------------|
| Skill frontmatter | Claude Code skills |
| Hook configuration | Claude Code hooks |
| Settings changes | Claude Code settings |
| Agent frontmatter | Claude Code sub-agents |
| Plugin config | Claude Code plugins |
| Memory/checkpoint | Claude Code memory |
| Model/effort config | Claude Code model configuration |
### Dry-Run Mode

When `--dry-run` is active, replace all file writes with previews:
- `WOULD WRITE:` for frontmatter additions
- `WOULD CHECK:` for verification-only items
- `WOULD CREATE:` for new files
- Never call Edit or Write tools
- `[DRY RUN]` label on every implementation block
- Batch uses `[~]` instead of `[x]`

### Live Mode

1. Make the changes (Edit tool)
2. **Verify** — re-read each modified file to confirm
3. Show verification result with `[next] [back] [rescan] [done]`

### Batch

Implement sequentially, verifying each. Show progress checklist.
After completion, show implementation summary. Read output-templates.md for format.

### Rescan

After implementation, rebuild adoption matrix. Mark fully-adopted items as DONE.

### Skip Behavior

- `skip` → move to next
- `skip all` / `back` → return to dashboard
- `stop` / `done` → exit implementation

---

## Phase 6: Ticket / Task Creation

When the user says `[tickets]`, detect what's available and offer options:

1. **If a `/tickets` skill exists** (any issue tracker — Linear, Jira, GitHub Issues, etc.):
   delegate to it. Group 3+ related items into epics. Show preview, ask `[create] [modify] [back]`.
2. **If `gh` CLI is available** but no `/tickets` skill:
   offer to create GitHub Issues directly: `gh issue create --title "..." --body "..."`
3. **If neither is available:**
   > "No issue tracker configured. You can:
   > [export]  Write items to a markdown file you can import later
   > [skip]    Skip ticket creation for now
   > [back]    Return to dashboard"

The user can also **update items interactively** without any external tool:
- Mark items as done: `done #3` or `mark #3 done` → updates the dashboard DONE section
- Skip items: `skip #5` → moves to "Skipped" with a reason prompt
- Add notes: `note #7 waiting for v2.2 to ship` → stored in checkpoint
- Change priority: `promote #10` → moves from Nice to Have to Useful tier

These in-session state changes are **auto-saved incrementally** — after each
`done`, `skip`, `note`, or `promote` action, immediately update the checkpoint
file on disk. This way, if the session ends unexpectedly (terminal closed, crash),
progress is not lost. The next `/whats-new` run picks up where they left off.

Auto-save writes only the "Actions taken" section of the checkpoint, not the
full checkpoint (which includes adoption matrix snapshots and is written on exit).

Read `${CLAUDE_SKILL_DIR}/references/advanced.md` for ticket creation format.

---

## Phase 7: The Loop

**CRITICAL:** After every action, return to the interactive menu.

**Before every menu**, re-display a context summary line so the user never has to scroll
up to understand their options. The context line varies by phase:

- **Dashboard menu**: `[N] actionable | [N] done | [N] skipped | Items: !! #1 | ** #2-#4 | >> #14-#16`
- **Drill-down menu**: `Viewing #[N] of [total] | Tier: [tier] | [N] done, [N] remaining`
- **Post-implementation**: `Implemented: #[N] | Remaining: [N] items | Quick wins left: [N]`
- **Post-batch summary**: `Session: [N] implemented, [N] remaining | Adoption: model [N]%, effort [N]%`

This is essential for usability — the user may be many screens away from the dashboard.

Exits ONLY when the user says: `exit`, `done`, `stop`, `quit`, `that's all`, `I'm done`

If ambiguous: "Anything else to drill into or implement? Or save checkpoint and exit?"

---

## Phase 8: Checkpoint

Only advance when the user has taken action. Browsing alone does not advance it.

**Dry-run:** Skip checkpoint save entirely. Show:
> "Dry run complete — no changes written, no checkpoint saved."

**Live mode exit — implemented items:** Recommend advancing.
**Live mode exit — browse only:** Ask explicitly (advance / keep / set to X.Y.Z).
**First run:** Always ask.

Write checkpoint to project memory directory. Always read from disk before writing.
Update MEMORY.md. Show exit banner. Suggest `/schedule weekly /whats-new` if not set up.

Read `${CLAUDE_SKILL_DIR}/references/output-templates.md` for checkpoint template and exit banner.

---

## Guidelines

- **Persistent numbering** — #1, #2... never change during a session
- **Never hardcode** repo names, paths, or skill names — discover dynamically
- **"reset back N"** means N changelog `## X.Y.Z` headings, not semver math. Confirm before writing
- **"show checkpoint" with none** → "No checkpoint set. Next run reviews everything."
- **All features adopted** → "Stack is fully up to date. Run `--full` to re-audit or wait for new versions."
- **Be educational** — explain *why*, not just *what*
- **Show, don't tell** — before/after code for non-trivial changes
- **Include rollback** — every drill-down shows how to undo
- **Cost-aware** — order-of-magnitude estimates where possible
- **Scope-aware** — always show whether a change is global or project-level
- **Verify after implementing** — re-read files to confirm changes landed
- **Rescan after batch** — rebuild adoption matrix to reflect current state
- **Track progress** — mark implemented items as DONE, don't hide them
- **Respect skip** — move on immediately, no follow-up
- **Don't re-recommend adopted features** — check frontmatter first
- **Flag version requirements** — if a feature needs a newer CC version, say so
- **Parse naturally** — plain English always works, flags are shortcuts
- **Loop until exit** — never end on your own; return to menu
- **Markdown tables only** — standard `| col |` + `|---|`. Box-drawing OK for banners only
- **Broad by default** — global unless user specifies a repo
- **Suggest `/schedule`** — on exit, hint at automated weekly runs
- **Checkpoint race** — last writer wins, acceptable. Read before writing.
- **Issue tracker agnostic** — detect `/tickets`, `gh`, or neither. Always offer
  `[export]` as a fallback. Users can manage items interactively without any tracker.
- **Composability** — if invoked from another skill, suppress interactive loop.
  Run Phase 0–3, output dashboard, return. Don't wait for input.
- **Token management** — read changelog selectively, not the entire file
