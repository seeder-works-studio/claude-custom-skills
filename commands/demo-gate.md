# /demo-gate

Add a full-site password gate to any React/Vite demo site. Prevents bots, scrapers, and phishing detectors from indexing content while giving real clients a clean, branded access experience.

## What this skill does

1. Reads the project's color palette from `src/index.css` (CSS custom properties)
2. Creates a `DemoGate` component matching the site's brand colors
3. Wraps the root `App` component so **nothing renders** until the correct password is entered
4. Adds a "PRIVATE CLIENT DEMO" banner explaining the site's purpose
5. Stores auth in `sessionStorage` (clears when the browser tab closes)

## When to use

- You're building a demo site for a client and it keeps getting flagged as phishing
- You want to prevent bots/scrapers/search engines from indexing demo content
- You need a quick password gate that looks professional and on-brand

---

## Instructions for Claude

### Step 0 — Check if DemoGate already exists

Before doing anything, check:

```bash
find src -name "DemoGate*"
grep -r "DemoGate\|demo_gate\|demo_auth" src/
```

If `DemoGate` is already present and imported in `App.tsx` or `main.tsx`, **stop and report** to the user:
> "DemoGate is already installed in this project — no changes needed."

Only proceed if it does not exist.

Read the project structure:

```
- src/App.tsx (or App.jsx) — find the root component to wrap
- src/main.tsx — check if wrapping here is cleaner (see Step 3)
- src/index.css — extract CSS custom property values for brand colors
- package.json — confirm it's a React project
```

Then implement the following:

### Step 1 — Extract brand colors

From `src/index.css`, find:
- `--background` → page background (light mode)
- `--primary` → primary brand color (use for dark background variant)
- `--accent` → accent/highlight color (use for banner + button)
- `--sidebar-background` → if present, use as the gate page background (usually darkest brand color)
- `--sidebar-border` → use for card border

If no CSS variables exist, default to:
- Gate background: `#0f2027`
- Accent: `#16a34a`
- Border: `#1e3a4a`

### Step 2 — Create `src/components/DemoGate.tsx`

Use inline styles (not Tailwind classes) so the gate renders correctly before any CSS context loads. The component must:

- Accept `children: ReactNode` and wrap them
- Check `sessionStorage.getItem('demo_gate_auth') === '1'` on mount
- If unlocked, render children immediately (no flash)
- If locked, render the gate page described below

**Gate page layout:**
```
┌─────────────────────────────────────────────────────┐  ← full viewport, dark brand background
│  [GREEN BANNER] PRIVATE CLIENT DEMO — confidential  │  ← fixed top, accent color
│                                                     │
│         ┌──────────────────────────────┐            │
│         │  [logo/icon]  Site Name      │            │  ← card, darker shade, brand border
│         │  Private Demo Access         │            │
│         │                              │            │
│         │  [explanation copy]          │            │
│         │                              │            │
│         │  ACCESS CODE                 │            │
│         │  [password input]            │            │
│         │  [Enter Demo button]         │            │
│         │                              │            │
│         │  ─────────────────────────── │            │
│         │  📊 Simulated data           │            │
│         │  🔒 Not indexed              │            │
│         │  🚫 Not a live system        │            │
│         └──────────────────────────────┘            │
│                                                     │
│  Prepared by [Studio Name] · Confidential preview   │
└─────────────────────────────────────────────────────┘
```

**Password:** Ask the user what password to use, or default to `demo2026` if they don't specify.

**Error state:** Shake animation on the card + red error message below the input. Clear the input on wrong entry.

**Shake keyframe:** Inject via a `<style>` tag inside the component:
```css
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  20%       { transform: translateX(-8px); }
  40%       { transform: translateX(8px); }
  60%       { transform: translateX(-5px); }
  80%       { transform: translateX(5px); }
}
```

**Copy to use (adapt to client):**
- Banner: `PRIVATE CLIENT DEMO — This site is a confidential preview prepared by [Studio]`
- Heading: `Private Demo Access`
- Body: `This is a confidential product demo built for your review. It contains simulated data representative of your operation. Enter the access code provided by your [Studio] contact to continue.`
- Bullets: workbook-backed / not indexed / not a live system

### Step 3 — Choose where to wrap

Read `src/App.tsx` first. Pick the right wrapping strategy:

**Option A — Single return in App.tsx (most common)**

App has one top-level JSX return. Wrap it directly:

```tsx
import DemoGate from "@/components/DemoGate";

const App = () => (
  <DemoGate>
    <QueryClientProvider ...>
      {/* rest of app */}
    </QueryClientProvider>
  </DemoGate>
);
```

**Option B — Multiple/conditional returns in App.tsx**

App has `if (!authed) return <LoginPage />; return <Dashboard />;` style logic. Wrapping inside App gets messy. Wrap in `main.tsx` instead:

```tsx
// main.tsx
import DemoGate from "./components/DemoGate.tsx";

createRoot(document.getElementById("root")!).render(
  <DemoGate>
    <App />
  </DemoGate>
);
```

In both cases the gate must be the **outermost** wrapper so nothing leaks before auth.

### Step 4 — Verify

Run `npm run build` (or `bun run build`) and confirm:
- No TypeScript errors
- Build succeeds
- `DemoGate` chunk appears in build output

### Step 5 — Commit

```
feat: add password-gated demo landing page

Prevents bots and scrapers from indexing demo content. Gate uses
brand colors, explains the site is a confidential preview, and
requires access code before any app content is rendered.
```

---

## Customization options

| Option | Default | How to change |
|--------|---------|---------------|
| Password | `demo2026` | Change `DEMO_PASSWORD` constant in `DemoGate.tsx` |
| Storage key | `demo_gate_auth` | Change `DEMO_KEY` constant |
| Session scope | Tab close clears auth | Switch `sessionStorage` → `localStorage` for persistent auth |
| Studio name | Seederworks Studio | Update banner + footer copy |
| Bullet points | 3 generic items | Adapt to client's data source and context |
| Logo mark | ⚡ emoji | Replace with `<img>` tag pointing to client logo |

---

## Example usage

```
/demo-gate
```

Claude will read the project colors, implement the gate, wrap App.tsx, build, and commit.

```
/demo-gate password=haadthip2026
```

Claude will use `haadthip2026` as the access code instead of the default.
