# claude-custom-skills

A collection of custom Claude Code slash commands for developer productivity.

## Installation

Copy any command from `commands/` to your `~/.claude/commands/` directory:

```bash
cp commands/dev-cost-estimate.md ~/.claude/commands/
```

Or clone and symlink the whole directory:

```bash
git clone https://github.com/seeder-works-studio/claude-custom-skills.git
ln -s $(pwd)/claude-custom-skills/commands/* ~/.claude/commands/
```

## Commands

### `/dev-cost-estimate [claude-hours]`

Analyzes your current project's git history and codebase to produce a professional development cost report showing:

- What it would cost to build this with a human team (Solo, Lean Startup, Growth Co, Enterprise)
- Speed multiplier vs. a human developer
- Value per Claude hour
- ROI of using Claude vs. hiring

**Usage:**
```
/dev-cost-estimate          # Claude estimates hours from git commits
/dev-cost-estimate 30       # Provide your actual Claude active hours
```

**Output includes:**
- Project snapshot (commits, LOC, tech stack, integrations)
- Human hours estimate with breakdown
- 4-tier team cost comparison table
- Value per Claude hour table
- Cost comparison and ROI
- Headline summary for sharing with clients/investors

**Rate assumptions (2025–2026 US market):**
- Solo contractor: $125/hr
- Growth Co blended: $150/hr (loaded)
- Enterprise blended: $175/hr (loaded)
- Claude Pro: $20/month ($0.67/day)

## Contributing

PRs welcome. Each command is a single `.md` file in `commands/`.
