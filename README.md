# claude-custom-skills

A collection of custom Claude Code slash commands for developer productivity.

## How it works

Claude Code automatically loads any `.md` file placed in `~/.claude/commands/` as a slash command. No config, no registration, no restart — just drop the file in and the command is live.

## Installation

### Option 1 — Copy a single skill

```bash
curl -o ~/.claude/commands/dev-cost-estimate.md \
  https://raw.githubusercontent.com/seeder-works-studio/claude-custom-skills/main/commands/dev-cost-estimate.md
```

### Option 2 — Clone and symlink all skills (recommended)

New skills you add to the repo instantly become available with no extra steps.

```bash
git clone https://github.com/seeder-works-studio/claude-custom-skills.git ~/claude-custom-skills
ln -s ~/claude-custom-skills/commands/* ~/.claude/commands/
```

### Option 3 — Manual copy

Download or copy any `.md` file from the `commands/` folder in this repo into `~/.claude/commands/`.

## Calling a skill

Open any project in Claude Code and type the slash command in the chat:

```
/dev-cost-estimate
```

or pass optional arguments:

```
/dev-cost-estimate 30
```

That's it. Claude will execute the skill against your current project.

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
