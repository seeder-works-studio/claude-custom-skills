# /demo-observability

Implement page view telemetry for any React/Vite demo site using the demo-observability logging layer.

## What this skill does

1. Creates an analytics utility (`src/lib/analytics.ts` or `src/utils/analytics.ts`) with a reusable fetch function pointing to the `demo-observability` collector.
2. Identifies the demo's name from `package.json` to use as the unique `demo` identifier.
3. Wires up page view tracking in `App.tsx` (typically by using `useLocation` from `react-router-dom`).
4. Logs `page_view` events dynamically as the user navigates.

## When to use

- You are building a new client demo or internal prototype and want traffic visibility in the centralized SeederWorks demo dashboard.
- You want to track conversions, CTA clicks, or errors without bringing in a heavy third-party SDK.

---

## Instructions for Claude

### Step 0 — Check if Analytics is already installed

Check if the file already exists:
```bash
find src -name "analytics.ts"
grep -r "demo-observability" src/
```

If it already exists, report to the user:
> "Analytics tracking is already installed in this project — no changes needed."
Stop if it exists.

### Step 1 — Determine Demo Name

Read `package.json` to find the `"name"` property. Use this value as the `demo` identifier in the fetch snippet. If the name is unavailable, default to a slugified version of the directory name.

### Step 2 — Create the Analytics Utility

Create `src/lib/analytics.ts` (if `src/lib` exists, otherwise `src/utils/analytics.ts`).

Write the following code exactly into the file, replacing `<PROJECT_NAME>` with the demo name from Step 1:

```typescript
export const trackEvent = (eventName: string, meta: Record<string, any> = {}) => {
  try {
    fetch("https://demo-observability.seederworks.com/collect", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        event: eventName,
        demo: "<PROJECT_NAME>",
        client: "internal",
        env: import.meta.env?.MODE || "development",
        url: window.location.href,
        hostname: window.location.hostname,
        path: window.location.pathname,
        meta,
      }),
    }).catch(error => {
      console.error("Failed to send analytics event fetch", error);
    });
  } catch (error) {
    console.error("Failed to send analytics event", error);
  }
};
```

### Step 3 — Integrate into App.tsx

Read `src/App.tsx` to find the `<BrowserRouter>` or routing logic.

Add a tracking component that uses `react-router-dom`'s `useLocation` to track page views automatically:

```tsx
import { useEffect } from "react";
import { useLocation } from "react-router-dom";
import { trackEvent } from "./lib/analytics"; // adjust path if necessary

function AnalyticsTracker() {
  const location = useLocation();

  useEffect(() => {
    trackEvent("page_view", {
      path: location.pathname,
      search: location.search
    });
  }, [location]);

  return null;
}
```

Then, inject `<AnalyticsTracker />` inside the `<BrowserRouter>` (or equivalent Router provider) in the main render tree so it can access the location context.

If the app doesn't use `react-router-dom`, simply call `trackEvent("page_view")` inside a top-level `useEffect` in `App.tsx` that runs once on mount.

### Step 4 — Verify

Run `npm run build` or `bun run build` to verify there are no TypeScript or syntax errors. Ensure `react-router-dom` is imported correctly if used.

### Step 5 — Commit

If the project uses git, commit the changes:

```
feat: integrate demo-observability analytics tracking

Adds an analytics utility to push page view and custom event telemetry 
to the centralized demo-observability worker. Wires up a router listener 
in App.tsx to automatically track navigation events.
```

---

## Example usage

```
/demo-observability
```

Claude will automatically add the `analytics.ts` utility, read the project name, update `App.tsx` to listen to route changes, verify the build, and commit the implementation.
