# /whats-new

Stay current with Claude Code releases by cross-referencing the changelog against your actual stack — skills, agents, hooks, MCP servers, and plugins. Surfaces what matters to **you**, not just what shipped.

## Quick start

```
/whats-new
```

That's it. The skill walks you through everything interactively.

## What it does

1. **Reads the changelog** — groups features by impact, not by version number
2. **Scans your stack** — discovers every skill, agent, hook, MCP server, and plugin you have
3. **Shows a dashboard** — ranks improvements into 5 tiers (Breaking, Must Have, Useful, Nice to Have, Quick Wins)
4. **Lets you act** — drill into items, implement changes, create tickets, or defer to next time
5. **Remembers where you left off** — checkpoint system so repeat runs only show what's new

## Installation

Copy the `whats-new/` directory into your Claude Code skills folder:

```bash
# Global (available in all projects)
cp -r whats-new/ ~/.claude/skills/whats-new/

# Or project-level (available only in one repo)
cp -r whats-new/ .claude/skills/whats-new/
```

The directory structure should look like:

```
~/.claude/skills/whats-new/
├── SKILL.md                        # Core skill (398 lines)
├── README.md                       # This file
└── references/
    ├── output-templates.md         # Banners, menus, dashboard formats
    └── advanced.md                 # Cross-repo comparison, ticket creation, composability
```

### Requirements

- **Claude Code** v2.1.0+ (earlier versions lack skill frontmatter support)
- **Model**: The skill uses `model: opus` by default for best results. If you don't have Opus access, edit the frontmatter in `SKILL.md` to `model: sonnet`
- **Changelog cache**: Run `/release-notes` once to populate `~/.claude/cache/changelog.md`. The skill will remind you if it's missing.

### Optional integrations

| Integration | What it enables | How to check |
|-------------|-----------------|--------------|
| `/tickets` skill | Create tickets in Linear, Jira, etc. | `ls ~/.claude/skills/tickets/` |
| `gh` CLI | Create GitHub Issues directly | `which gh` |
| Neither | Export items to a markdown file | Always available |

## Usage

### Basic

```
/whats-new                          # Full review (checkpoint-aware)
/whats-new last 3 versions          # Only recent changes
/whats-new just hooks               # Focus on one area
/whats-new for my-project           # Target a specific repo
/whats-new --dry-run                # Preview without writing any files
```

### At the dashboard

```
3                                   # Drill into item #3
batch                               # Implement all Quick Wins at once
implement 2,4                       # Implement specific items
tickets 5-8                         # Create tickets for items 5-8
done #3                             # Mark item as done (no tracker needed)
skip #5                             # Skip with a reason
rescan                              # Refresh after manual changes
exit                                # Save checkpoint and exit
```

### Checkpoint management

The checkpoint tracks what you've **acted on**, not just what you've read.
Browsing alone doesn't advance it.

```
/whats-new show checkpoint          # See current state
/whats-new reset checkpoint to 2.1.70   # Re-review from a version
/whats-new reset checkpoint back 3      # Go back 3 versions
/whats-new clear checkpoint             # Start fresh next time
```

### Dry run

Preview everything without writing any files or saving a checkpoint:

```
/whats-new --dry-run                # Full preview
/whats-new --dry-run implement #1   # See what would change
/whats-new --dry-run batch          # Preview all Quick Wins
```

### Cross-repo comparison

```
/whats-new --compare repo-a repo-b  # Compare specific repos
/whats-new compare all repos        # Compare all repos in ~/Developer
```

## How the dashboard works

Items are categorized into 5 tiers:

| Tier | Label | What it means |
|------|-------|---------------|
| `!!` | Breaking / Urgent | Deprecations, security fixes affecting your stack |
| `**` | Must Have | Fixes pain points, saves significant cost |
| `++` | Useful | Meaningful workflow improvements |
| `--` | Nice to Have | Small quality-of-life, can defer |
| `>>` | Quick Wins | One-line changes, under 5 minutes |

Each item includes:
- **Why it matters to your stack** (names specific skills/agents affected)
- **Before/after code** showing the exact change
- **Cost impact** (order-of-magnitude estimates where applicable)
- **Synergies** with other items
- **Rollback** instructions

## Managing items without a tracker

You don't need Linear, Jira, or any external tool. Manage items directly:

```
done #3                             # Mark as done
skip #5                             # Skip (prompts for reason)
note #7 waiting for v2.2            # Add a note
promote #10                         # Move up one tier
```

These changes auto-save to disk incrementally — if your session crashes, progress is preserved.

## How the checkpoint works

- Each run **re-scans your actual frontmatter** — never trusts prior claims
- If an implementation was reverted (git reset, branch deleted), it reappears on the dashboard
- In-session actions (`done`, `skip`, `note`) auto-save to disk immediately
- The checkpoint version only advances when you confirm on exit
- Dry runs never save a checkpoint

## Composability

| With | What happens |
|------|-------------|
| `/schedule weekly /whats-new` | Automated weekly check-ins |
| `/implement` | Non-trivial items delegate to worktree isolation + PR |
| `/tickets` | Ticket creation with epic grouping for 3+ related items |
| Other skills calling `/whats-new` | Suppresses interactive loop, outputs dashboard, returns |

## Customization

### Change the model

Edit `SKILL.md` frontmatter:

```yaml
---
model: sonnet    # or haiku for faster/cheaper runs
effort: medium   # low, medium, high
---
```

### Change the scope

The skill discovers your stack dynamically. It scans:
- `~/.claude/skills/` and `~/.claude/agents/` (global)
- `$PWD/.claude/skills/` and `$PWD/.claude/agents/` (project)
- `~/.claude/settings.json` (hooks, MCP servers)
- `~/.claude/plugins/` (plugins)

No configuration needed — just have skills/agents/hooks and the skill finds them.

## FAQ

**Q: What if I don't have any skills or agents yet?**
The skill still works — it shows the changelog summary and recommends which features to adopt. The dashboard will focus on hooks, settings, and general improvements.

**Q: Does it modify my files?**
Only when you explicitly say `implement`. Use `--dry-run` to preview first. Every change shows a before/after diff and can be rolled back.

**Q: What happens if I close the terminal mid-session?**
In-session actions (`done`, `skip`, `note`, `promote`) auto-save to disk. Implementations that completed are already written. The only thing lost is the full adoption snapshot (written on clean exit).

**Q: Can I use this in CI/CD?**
Not directly — the skill is interactive. For automation, consider `/schedule weekly /whats-new` which runs as a remote agent.

**Q: I ran it twice and got different item numbers. Why?**
Item numbers are assigned fresh each run based on the current adoption matrix. If you implemented items between runs, the numbering shifts. Within a single session, numbers are stable.
