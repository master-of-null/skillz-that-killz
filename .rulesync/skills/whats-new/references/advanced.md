# Advanced Features

Reference file for `/whats-new` — load when user requests cross-repo comparison,
pain-point detection, or adoption history.

## Cross-Repo Comparison

When invoked with `--compare` or "compare my repos":

```
/whats-new --compare ~/Developer/repo-a ~/Developer/repo-b ~/Developer/repo-c
/whats-new compare all repos in ~/Developer
```

Scan each repo's `.claude/` config and produce a comparison matrix.
If only 1 repo has `.claude/` config, say so and offer to run a targeted analysis
on that repo instead: "Only one repo has .claude/ config. Run `/whats-new
for my-project` instead? [yes / exit]"

```
╔══════════════════════════════════════════════════════════════════════════╗
║                    CROSS-REPO ADOPTION MATRIX                           ║
╚══════════════════════════════════════════════════════════════════════════╝

| Feature           | repo-a (18 skills) | repo-b (5 skills) | repo-c (12 skills) |
|-------------------|--------------------|-------------------|--------------------|
| effort            | 4/18 (22%)         | 0/5 (0%)          | 8/12 (67%)         |
| model             | 6/18 (33%)         | 2/5 (40%)         | 12/12 (100%)       |
| context: fork     | 3/18 (17%)         | 0/5 (0%)          | 4/12 (33%)         |
| memory (agents)   | 3/3 (100%)         | 0/1 (0%)          | 1/2 (50%)          |
| hooks (scoped)    | 0/18 (0%)          | 0/5 (0%)          | 0/12 (0%)          |

Most behind: repo-b (0% adoption across 3 features)
Most current: repo-c (67% average adoption)
```

**ASK:** "Want to drill into a specific repo, or apply improvements across all?"

## Session Pain-Point Detection

When building recommendations, optionally scan for signals of actual pain in recent sessions.
This makes recommendations **data-driven** rather than just config-driven.

Look for these signals in the current project's session history
(`~/.claude/projects/<project-key>/`):

| Signal | Where to Find | What it Suggests |
|--------|--------------|-------------------|
| Frequent compaction | Session JSONL — many `compact` entries | Add `context: fork` to heavy skills |
| Rate limit errors | Session JSONL — `StopFailure` or 429 mentions | Add `StopFailure` hook, batch API calls |
| Long-running skills | Turn duration data in sessions | Add `model: haiku/sonnet` for quick skills |
| Repeated permission prompts | Session JSONL — many permission asks | Review `allowed-tools`, add `Bash()` rules |
| Memory warnings | `/context` output mentions | Add `effort: low` to simple skills |

Present detected pain points as a special section before the main dashboard:

```
━━━ DETECTED PAIN POINTS (from recent sessions) ━━━━━━━━━━━━━━━━━━━━━━

| Signal                        | Frequency    | Related Item |
|-------------------------------|-------------|--------------|
| Context compacted 12x in 3 sessions | High   | → #4 (fork)  |
| Rate limit hit 3x this week   | Medium      | → #8 (StopFailure) |

These items are automatically promoted to Must Have tier.
```

**Note:** Session scanning is best-effort. If sessions are large or numerous, sample the
most recent 3-5 sessions rather than scanning everything. If no signals are found, skip
this section silently.

## Adoption History Tracking

The checkpoint tracks adoption percentages at each review. Over multiple runs, this
enables trend reporting:

```
━━━ ADOPTION TREND ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Feature       | 2026-03-01 | 2026-03-15 | 2026-03-23 (now) |
|---------------|------------|------------|-------------------|
| effort        | 0%         | 22%        | 100%              |
| model         | 0%         | 33%        | 100%              |
| context: fork | 0%         | 17%        | 44%               |
```

To enable this, the checkpoint (Phase 8) saves the full adoption matrix on every run.
On subsequent runs, read the previous checkpoint and compare. If this is the first run,
show "baseline" instead of a trend.

## Composability

### With `/schedule`

```
/schedule weekly /whats-new --since last-checkpoint
```

Suggest this to the user on exit if they haven't set it up:
> "Tip: Run `/schedule weekly /whats-new` to get notified of new features automatically."

### With `/implement`

When implementing non-trivial items (not just frontmatter changes), delegate
to `/implement` for proper worktree isolation, PR creation, and adversarial review.

### With issue trackers (Linear, GitHub Issues, Jira, etc.)

Ticket creation is **tracker-agnostic**. Detect what's available:

1. **`/tickets` skill exists** → delegate to it (handles Linear, Jira, etc. per user config)
2. **`gh` CLI available** → offer GitHub Issues: `gh issue create --title "..." --body "..."`
3. **Neither** → offer `[export]` to write a markdown file

Group 3+ related items into epics/parent issues when the tracker supports it.

### Ticket creation example

```
Creating tickets for 5 items:

| #  | Item                       | Type        | Priority | Scope   |
|----|----------------------------|-------------|----------|---------|
| 5  | hooks in skill frontmatter | Sub-issue   | High     | global  |
| 6  | once: true for hooks       | Sub-issue   | Medium   | global  |

Epic: "Adopt Claude Code 2.x features" (items 5-8)
Standalone: item 9
```

**ASK:** "Look good? [create] [modify] [back]"

### Export fallback (no tracker configured)

Write a markdown file to the current working directory (`$PWD/whats-new-items.md`):

```markdown
# /whats-new Improvement Items (exported [date])

## Must implement
- [ ] #5: hooks in skill frontmatter (High, ~30 min)
- [ ] #6: once: true for hooks (Medium, ~10 min)

## Nice to have
- [ ] #10: PostCompact hook (Low, ~20 min)
```

### Interactive item management (no external tool needed)

Users can manage items directly in the dashboard without any tracker:
- `done #3` → mark as done, moves to DONE section
- `skip #5` → prompt for reason, moves to Skipped in checkpoint
- `note #7 waiting for v2.2` → attach a note, stored in checkpoint
- `promote #10` → move up one tier (Nice to Have → Useful)
