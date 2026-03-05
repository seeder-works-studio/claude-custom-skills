---
description: Analyze the current project and produce a professional dev cost estimate report — showing what it would cost to build this with a human team vs. Claude, ROI, speed multiplier, and value per Claude hour. Optionally accepts Claude hours as an argument (e.g. /dev-cost-estimate 30).
---

# Dev Cost Estimate

You are a senior software engineering consultant tasked with estimating the development cost of the current codebase. Produce a comprehensive, professional report benchmarking this work against real-world hiring costs, then calculate the ROI of using Claude.

---

## Step 1 — Analyze the Codebase

Use Glob and Read tools to systematically review the project. Understand:
- Total lines of code by language
- Architectural complexity (frameworks, integrations, APIs)
- Advanced features (real-time, GPU, native plugins, streaming, etc.)
- Testing coverage
- Documentation quality

Run these shell commands for baseline metrics:

```bash
# Project identity
git remote get-url origin 2>/dev/null || basename $(pwd)
cat package.json 2>/dev/null | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('name',''))" 2>/dev/null

# Commit history (skip init/scaffold commits for first date)
git log --oneline | wc -l
git log --reverse --pretty=format:"%ad %s" --date=short \
  | grep -viE '^\S+ (init|initial commit|initial|scaffold|create repo|first commit|add readme|repo init)$' \
  | head -1
git log -1 --pretty=format:"%ad" --date=short

# Code volume
git diff --stat $(git rev-list --max-parents=0 HEAD) HEAD 2>/dev/null | tail -1
git ls-files | wc -l

# Language breakdown
git ls-files | grep -E '\.(ts|tsx)$' | wc -l
git ls-files | grep -E '\.(js|jsx|mjs)$' | wc -l
git ls-files | grep -E '\.py$' | wc -l
git ls-files | grep -E '\.(go)$' | wc -l
git ls-files | grep -E '\.(rs)$' | wc -l
git ls-files | grep -E '\.(swift)$' | wc -l
git ls-files | grep -E '\.(java|kt)$' | wc -l
git ls-files | grep -E '\.(cs)$' | wc -l
git ls-files | grep -E '\.(cpp|cc|c|h)$' | wc -l
git ls-files | grep -E '\.(metal|glsl|wgsl|hlsl)$' | wc -l
git ls-files | grep -E '\.(sql|prisma)$' | wc -l
git ls-files | grep -E '\.(test|spec)\.(ts|tsx|js|py|go|rs)$' | wc -l
git ls-files | grep -E '\.(md|txt)$' | wc -l

# Complexity signals
git ls-files | grep -iE '(api|route|controller|service|hook|component|migration|schema|worker|webhook)' | wc -l
ls -d */ 2>/dev/null | head -20
```

Also scan for integrations and advanced features by reading key config files (`package.json`, `requirements.txt`, `go.mod`, `Cargo.toml`, `*.tf`, `docker-compose.*`, CI config files, etc.) and a sample of source files.

---

## Step 2 — Calculate Development Hours

Use these **hourly productivity estimates** for a senior developer (5+ years experience):

| Code Type | Lines/Hour | Examples |
|-----------|-----------|---------|
| Simple CRUD / UI components | 40–50 | forms, list views, basic REST |
| Standard business logic | 25–35 | workflows, data transforms |
| Complex business logic | 20–30 | multi-step flows, state machines |
| Auth systems (JWT, OAuth, SSO) | 15–25 | login, sessions, permissions |
| External API integrations | 15–20 | per integration, inc. error handling |
| Real-time features (WS, SSE) | 15–20 | live updates, streaming |
| Database schema + migrations | 20–30 | models, indexes, relations |
| GPU / shader programming | 10–20 | Metal, GLSL, WGSL |
| Native C/C++ interop | 10–20 | FFI, plugins, extensions |
| Video/audio/media processing | 10–15 | encoding, capture, playback |
| System extensions / plugins | 8–12 | OS-level, daemon, kernel adjacent |
| Infrastructure as code | 20–30 | Terraform, Docker, CI/CD |
| Comprehensive tests | 25–40 | unit, integration, e2e |

**Overhead multipliers** (applied to base coding hours):

| Activity | Overhead |
|----------|---------|
| Architecture & design | +15–20% |
| Debugging & troubleshooting | +25–30% |
| Code review & refactoring | +10–15% |
| Documentation | +10–15% |
| Integration & testing | +20–25% |
| Learning curve (new/specialized tech) | +10–20% |

Calculate total hours:
1. Classify files/LOC by type from Step 1
2. Apply the matching productivity rate
3. Sum base hours, then apply overhead multipliers
4. Show the full breakdown explicitly

---

## Step 3 — Research Current Market Rates

Use WebSearch to find actual 2025 rates. Search:
- `"senior full stack developer hourly rate 2025"`
- `"senior software engineer contractor rate 2025 United States"`
- `"[primary language from Step 1] developer freelance rate 2025"` (e.g. "Swift developer", "Go developer")

Use real results to set the rate range. If WebSearch is unavailable, use these 2025 US market defaults:
- **Low end**: $100–110/hr (remote, mid-level market)
- **Average**: $125–140/hr (standard US senior)
- **High end**: $175–225/hr (SF/NYC or highly specialized: GPU, native OS, embedded)

Pick a **recommended rate** based on the tech stack detected. Specialized stacks (Metal, CoreMediaIO, Rust, WASM, embedded) command the high end.

---

## Step 4 — Apply Organizational Overhead

Real developers don't code 40 hrs/week. Convert raw dev hours into realistic calendar time.

**Weekly time allocation by company type:**

| Company Type | Coding Efficiency | Coding Hrs/Week | Notes |
|-------------|-------------------|-----------------|-------|
| Solo / Founder | 70–80% | 28–32 hrs | No meetings overhead |
| Lean Startup | 60–70% | 24–28 hrs | Some standups, minimal process |
| Growth Company | 50–60% | 20–24 hrs | Standups, sprints, reviews, Slack |
| Enterprise | 40–50% | 16–20 hrs | Full ceremonies + governance |
| Large Bureaucracy | 30–40% | 12–16 hrs | Heavy process, compliance |

**Overhead sources eating the remaining time:**
- Daily standups (15 min × 5): 1.25 hrs/week
- Team syncs, all-hands: 1–2 hrs/week
- 1:1s with manager: 0.5–1 hr/week
- Sprint planning / retro: 1–2 hrs/week
- Code reviews (giving): 2–3 hrs/week
- Slack / email / async comms: 3–5 hrs/week
- Context switching / interruptions: 2–4 hrs/week
- Admin, tooling, access requests: 1–2 hrs/week

**Calendar weeks formula:**
```
Calendar Weeks = Total Dev Hours ÷ (Coding Hrs/Week for that company type)
```

---

## Step 5 — Calculate Full Team Cost

Engineering doesn't ship products alone. Include supporting roles.

**Supporting role ratios** (as a fraction of engineering hours):

| Role | Ratio to Eng Hours | Typical Rate |
|------|-------------------|--------------|
| Product Management | 0.25–0.40× | $125–200/hr |
| UX/UI Design | 0.20–0.35× | $100–175/hr |
| Engineering Management | 0.12–0.20× | $150–225/hr |
| QA / Testing | 0.15–0.25× | $75–125/hr |
| Project / Program Management | 0.08–0.15× | $100–150/hr |
| Technical Writing | 0.05–0.10× | $75–125/hr |
| DevOps / Platform | 0.10–0.20× | $125–200/hr |

**Team composition by stage:**

| Stage | PM | Design | EM | QA | PgM | Docs | DevOps | Multiplier |
|-------|-----|--------|-----|-----|------|------|--------|------------|
| Solo/Founder | 0% | 0% | 0% | 0% | 0% | 0% | 0% | **1.0×** |
| Lean Startup | 15% | 15% | 5% | 5% | 0% | 0% | 5% | **~1.45×** |
| Growth Company | 30% | 25% | 15% | 20% | 10% | 5% | 15% | **~2.2×** |
| Enterprise | 40% | 35% | 20% | 25% | 15% | 10% | 20% | **~2.65×** |

```
Full Team Cost = Engineering Cost × Team Multiplier
```

---

## Step 6 — Determine Claude Active Hours

If the user passed a number as `$ARGUMENTS` (e.g. `/dev-cost-estimate 30`), use that as Claude active hours.

Otherwise, estimate from git commit timestamps:

```bash
git log --format="%ai" | sort
```

**Session clustering method:**
1. Sort all commit timestamps chronologically
2. Group commits within a **4-hour window** as one session
3. Estimate session duration by commit density:
   - 1–2 commits in window → ~1 hr
   - 3–5 commits → ~2 hrs
   - 6–10 commits → ~3 hrs
   - 10+ commits → ~4 hrs
4. Sum all session estimates

**Fallback** (no reliable timestamps): `Claude hours ≈ Total LOC ÷ 350`

State clearly which method was used.

---

## Step 7 — Generate the Report

Output the full report in this format:

---

```
=====================================================================
  DEV COST ESTIMATE REPORT
  Project:   [name from package.json / go.mod / git remote / folder]
  Generated: [today's date]
=====================================================================

PROJECT SNAPSHOT
────────────────────────────────────────────────────────────────────
  Repository:     [git remote or folder name]
  First commit:   [date of first substantive commit]
  Last commit:    [date]
  Calendar span:  [X days / X weeks / X months]
  Total commits:  [N]
  Files tracked:  [N]
  Lines of code:  [N total insertions]

  Language breakdown:
    [Language]:   [N] lines
    [Language]:   [N] lines
    Tests:        [N] lines
    Docs/Config:  [N] lines

  Key tech:       [frameworks, runtimes, ORMs detected]
  Integrations:   [external APIs, services, OAuth providers]
  Advanced:       [real-time, GPU, native, streaming, etc. if present]

DEVELOPMENT HOURS ESTIMATE
────────────────────────────────────────────────────────────────────
  Code Type Breakdown:
    [Type] ([N] lines ÷ [rate] lines/hr):   [N] hrs
    [Type] ([N] lines ÷ [rate] lines/hr):   [N] hrs
    ...
  ─────────────────────────────────────────
  Base coding hours:                        [N] hrs

  Overhead:
    Architecture & design (+[X]%):          [N] hrs
    Debugging & troubleshooting (+[X]%):    [N] hrs
    Code review & refactoring (+[X]%):      [N] hrs
    Documentation (+[X]%):                  [N] hrs
    Integration & testing (+[X]%):          [N] hrs
    Learning curve (+[X]%):                 [N] hrs
  ─────────────────────────────────────────
  Total estimated dev hours:                [N] hrs

MARKET RATE RESEARCH (2025)
────────────────────────────────────────────────────────────────────
  Low end:    $[X]/hr  (remote, standard market)
  Average:    $[X]/hr  (US senior developer)
  High end:   $[X]/hr  ([location / specialization justification])
  Recommended: $[X]/hr  ([rationale based on tech stack])

ENGINEERING COST (SOLO BASELINE)
────────────────────────────────────────────────────────────────────
  Scenario     Rate      Hours   Total Cost
  ─────────────────────────────────────────
  Low-end      $[X]/hr   [N]     $[X]
  Average      $[X]/hr   [N]     $[X]
  High-end     $[X]/hr   [N]     $[X]

  Recommended range: $[X] – $[X]

REALISTIC CALENDAR TIME (with organizational overhead)
────────────────────────────────────────────────────────────────────
  Company Type       Efficiency   Coding Hrs/Wk   Calendar Time
  ─────────────────────────────────────────────────────────────
  Solo / Founder     75%          30 hrs           ~[N] months
  Lean Startup       65%          26 hrs           ~[N] months
  Growth Company     55%          22 hrs           ~[N] months
  Enterprise         45%          18 hrs           ~[N] months

FULL TEAM COST (all roles)
────────────────────────────────────────────────────────────────────
  Role Breakdown (Growth Company example):
    Role                  Hours     Rate       Cost
    ─────────────────────────────────────────────────
    Engineering           [N] hrs   $[X]/hr    $[X]
    Product Management    [N] hrs   $[X]/hr    $[X]
    UX/UI Design          [N] hrs   $[X]/hr    $[X]
    Engineering Mgmt      [N] hrs   $[X]/hr    $[X]
    QA / Testing          [N] hrs   $[X]/hr    $[X]
    Project Management    [N] hrs   $[X]/hr    $[X]
    Technical Writing     [N] hrs   $[X]/hr    $[X]
    DevOps / Platform     [N] hrs   $[X]/hr    $[X]
    ─────────────────────────────────────────────────
    TOTAL                 [N] hrs              $[X]

GRAND TOTAL SUMMARY
────────────────────────────────────────────────────────────────────
  Metric               Solo        Lean Startup  Growth Co   Enterprise
  ──────────────────────────────────────────────────────────────────
  Calendar Time        ~[N] mo     ~[N] mo       ~[N] mo     ~[N] yrs
  Total Human Hours    [N]         [N]           [N]         [N]
  Total Cost           $[N]K       $[N]K         $[N]K       $[N]K

CLAUDE ROI ANALYSIS
────────────────────────────────────────────────────────────────────
  Claude active hours:  [N] hrs  ([method: user-provided / git session clustering / LOC estimate])
  Sessions identified:  [N] sessions across [N] calendar days

  Value per Claude Hour:
    Value Basis             Total Value    Claude Hrs    $/Claude Hour
    ─────────────────────────────────────────────────────────────────
    Engineering only (avg)  $[X]           [N] hrs       $[X,XXX]/hr
    Full team (Growth Co)   $[X]           [N] hrs       $[X,XXX]/hr

  Speed vs. Human Developer:
    Estimated human hours for same work:  [N] hrs
    Claude active hours:                  [N] hrs
    Speed multiplier:                     [N]×  (Claude was [N]× faster)

  Cost Comparison:
    Human developer cost:    $[X]  (at $[X]/hr × [N] hrs)
    Estimated Claude cost:   ~$[X]  ([N] days × $0.67/day pro-rated)
    Net savings:             ~$[X]
    ROI:                     ~[N]×  (every $1 on Claude → $[N] of value)

THE HEADLINE
────────────────────────────────────────────────────────────────────
  Claude worked for approximately [N] hours across [N] calendar days
  and produced the equivalent of $[X] in professional engineering
  value — roughly $[X,XXX] per Claude hour. A growth-stage company
  would spend $[X] and [N] months to build this with a full team.
────────────────────────────────────────────────────────────────────

ASSUMPTIONS
────────────────────────────────────────────────────────────────────
  1. Rates based on US market averages (2025–2026)
  2. Senior developer (5+ years experience) — see rate research above
  3. Organizational overhead per company type as documented
  4. Claude Pro subscription: $20/month ($0.67/day), pro-rated
  5. Calendar time counted from first substantive commit (not git init)
  6. Does not include: marketing, legal, hosting/infrastructure,
     or ongoing maintenance post-launch
  7. [Add project-specific notes: specialized tech premium, domain
     expertise required, compliance needs, etc.]
```

---

After the report, add a brief **"How to Use This"** section:

> **Sharing this estimate:** Useful for client proposals, investor conversations, team retrospectives, or just appreciating what Claude built. Numbers use conservative US market rates — offshore rates show even higher multipliers.
>
> **Caveats:** Human hour estimates assume a single developer working sequentially. A strong team with existing tooling may move faster in some areas. Claude hours represent active coding sessions, not idle/thinking time. Rate research reflects market conditions at time of generation — verify against current job postings for high-stakes decisions.
