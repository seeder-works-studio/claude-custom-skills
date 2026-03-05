---
description: Analyze the current project and produce a professional dev cost estimate report — showing what it would cost to build this with a human team vs. Claude, ROI, speed multiplier, and value per Claude hour. Optionally accepts Claude hours as an argument (e.g. /dev-cost-estimate 30).
---

# Dev Cost Estimate

You are producing a professional development cost report for the current project. This report benchmarks the work done against real-world hiring costs across four team configurations, then calculates the ROI of using Claude.

## Step 1 — Gather Project Data

Run these commands to understand the codebase:

```bash
git log --oneline | wc -l                          # total commits

# First SUBSTANTIVE commit — skip init/initial/scaffold commits that have no real code
git log --reverse --pretty=format:"%ad %s" --date=short | grep -viE '^\S+ (init|initial commit|initial|scaffold|create repo|first commit|add readme|repo init)$' | head -1

# Last commit date
git log -1 --pretty=format:"%ad" --date=short

git diff --stat $(git rev-list --max-parents=0 HEAD) HEAD 2>/dev/null | tail -1  # total insertions/deletions
git ls-files | wc -l                               # total tracked files
git ls-files | grep -E '\.(ts|tsx|js|jsx)$' | wc -l   # TS/JS files
git ls-files | grep -E '\.(py|go|rs|java|rb|php|cs)$' | wc -l  # backend files
git ls-files | grep -E '\.(sql|prisma)$' | wc -l  # database files
git ls-files | grep -E '\.(md|txt|json|yaml|yml|toml)$' | wc -l  # config/docs
cat package.json 2>/dev/null | python3 -c "import sys,json; d=json.load(sys.stdin); deps=list(d.get('dependencies',{}).keys())+list(d.get('devDependencies',{}).keys()); print(len(deps),'dependencies')" 2>/dev/null || echo "no package.json"
```

Also check for architectural complexity signals:
```bash
git ls-files | grep -iE '(api|route|controller|service|hook|component|migration|schema|test|spec)' | wc -l
ls -d */ 2>/dev/null | head -20  # top-level directories
```

## Step 2 — Determine Claude Hours

If the user provided a number as an argument (e.g. `/dev-cost-estimate 30`), use `$ARGUMENTS` as the Claude active hours.

Otherwise, estimate Claude hours from git commit timestamps:
```bash
git log --pretty=format:"%ad" --date=format:"%Y-%m-%d %H:%M" | head -100
```
Group commits into sessions (any gap > 2 hours = new session). Estimate 1–4 hours per session based on commit density. State your assumption clearly.

## Step 3 — Estimate Human Hours

Use this rubric based on Lines of Code (LOC) and complexity signals:

| Complexity Tier | LOC Range | Files | Human hrs/LOC | Notes |
|----------------|-----------|-------|----------------|-------|
| Simple CRUD | <2,000 | <30 | 0.15 | forms, basic API |
| Moderate App | 2,000–8,000 | 30–100 | 0.12 | auth, integrations, DB |
| Complex System | 8,000–20,000 | 100–300 | 0.10 | multi-service, real-time |
| Enterprise | 20,000+ | 300+ | 0.08 | distributed, compliance |

Adjust upward (+20–40%) for:
- External API integrations (each one: +40 hrs)
- Auth systems (SSO, OAuth, JWT): +60 hrs
- Real-time features (WebSockets, streaming): +80 hrs
- Payment/billing systems: +100 hrs
- Database migrations with existing data: +40 hrs
- CI/CD pipelines: +30 hrs
- Multi-environment deployments: +20 hrs

Show your calculation explicitly.

## Step 4 — Calculate Costs

Use these 2025–2026 US market rate benchmarks:

**Solo Developer** (1 person, full-stack contractor)
- Rate: $125/hr
- Calendar multiplier: 1.0x (works alone, sequential)

**Lean Startup** (2 people: 1 senior + 1 mid-level)
- Blended rate: $125/hr avg
- Total hours × 1.45 (coordination overhead + context switching)
- Calendar: parallel work reduces calendar time ~25%

**Growth Co** (4–6 person team: PM, 2 seniors, 1 mid, QA, designer)
- Loaded cost: $150/hr blended (includes benefits, overhead)
- Total hours × 2.2 (meetings, PRs, reviews, QA cycles, PM time)
- Calendar: 22–26 months typical for complex apps

**Enterprise** (full team + compliance + architecture review)
- Loaded cost: $175/hr blended
- Total hours × 2.65 (governance, security review, docs, audits)
- Calendar: 2.5–3.5 years

**Claude cost** (Pro subscription): $20/month. Prorate to calendar days used.
`claude_cost = (calendar_days / 30) * 20`
Use calendar days from first substantive commit to last commit — not from `git init`.

## Step 5 — Output the Report

Format the report exactly as shown below. Use actual numbers from your analysis.

---

```
====================================================
  DEV COST ESTIMATE REPORT
  Project: [project name from package.json or git remote]
  Generated: [today's date]
====================================================

PROJECT SNAPSHOT
----------------
  Repository:     [git remote origin or folder name]
  First commit:   [date of first substantive commit — skip init/scaffold commits]
  Last commit:    [date]
  Calendar span:  [X days / X weeks / X months, calculated from first substantive commit]
  Total commits:  [N]
  Files tracked:  [N]
  Lines of code:  [N insertions from git diff stat]
  Key tech:       [list top frameworks/tools detected]
  Integrations:   [list external APIs/services found]

HUMAN HOURS ESTIMATE
--------------------
  Base LOC hours:       [N] hrs  ([LOC] lines × [rate])
  Integration premium:  [N] hrs  ([list what added hours]
  Auth/security:        [N] hrs
  DevOps/CI-CD:         [N] hrs
  ─────────────────────────────
  Total human hours:    [N] hrs  (solo baseline)

CLAUDE ACTIVE HOURS
-------------------
  Claude hours used:    [N] hrs  ([source: user-provided / estimated from commits])
  Speed multiplier:     [Nx]  (Claude was [N]x faster than a solo human dev)

VALUE PER CLAUDE HOUR
─────────────────────────────────────────────────────
  Value Basis           Total Value    Claude Hrs    $/Claude Hr
  ─────────────────────────────────────────────────────
  Engineering only      $[solo cost]   [N] hrs       $[rate]/hr
  Full team (Growth Co) $[growthco]    [N] hrs       $[rate]/hr
  ─────────────────────────────────────────────────────

GRAND TOTAL SUMMARY
─────────────────────────────────────────────────────────────────
  Metric              Solo        Lean Startup  Growth Co   Enterprise
  ─────────────────────────────────────────────────────────────────
  Calendar Time       ~[N] mo     ~[N] mo       ~[N] mo     ~[N] yrs
  Total Human Hours   [N]         [N]           [N]         [N]
  Total Cost          $[N]K       $[N]K         $[N]K       $[N]K
  ─────────────────────────────────────────────────────────────────

COST COMPARISON: CLAUDE vs. HUMAN
-----------------------------------
  Human developer cost (solo):    $[N]  (at $125/hr × [N] hrs)
  Estimated Claude cost:          ~$[N]  ([N] days × $0.67/day)
  Net savings:                    ~$[N]
  ROI:                            ~[N]x  (every $1 on Claude → $[N] of value)

THE HEADLINE
────────────────────────────────────────────────────────────────────
  Claude worked for approximately [N] hours across [N] calendar days
  and produced the equivalent of $[solo_cost] in professional
  engineering value — roughly $[value_per_hour] per Claude hour.
  A growth-stage company would spend $[growth_cost] and [N] months
  to build this with a full team.
────────────────────────────────────────────────────────────────────

ASSUMPTIONS
-----------
  1. Rates based on US market averages (2025–2026)
  2. Senior full-stack developer (5+ years experience) at $125/hr
  3. Growth Co loaded rate $150/hr (includes benefits, overhead, PM)
  4. Enterprise rate $175/hr (compliance, governance, architecture)
  5. Claude Pro subscription: $20/month ($0.67/day)
  6. Calendar time = human hours ÷ 160 hrs/month (full-time)
  7. No test coverage unless detected — production build adds ~15–20%
  8. Does not include: marketing, legal, hosting/infrastructure,
     or ongoing maintenance
  9. [Add any project-specific notes about integrations or complexity]
```

---

After the report, add a brief **"How to Use This"** section:

> **Sharing this estimate:** This report is useful for client proposals, investor conversations, retrospectives, or just appreciating what Claude built. The numbers use conservative US market rates — offshore rates would show even higher multipliers.
>
> **Caveats:** Human estimates assume sequential work. A skilled team with existing frameworks might be faster. Claude hours assume active coding sessions, not idle time.
