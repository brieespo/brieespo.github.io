# Personal Hub — Planning Doc / CLAUDE.md

Planning document for Bri's hub site: the dashboard that sits above her suite of personal apps (dinner planner, perfume tracker, law school command center, sewing tracker, restock). Drop into a new repo as `CLAUDE.md`. **Build this LAST** — after at least 2–3 apps exist with real data.

## The one rule that makes this work

**The hub owns no app data.** Every app writes to its own table in the shared Supabase project; the hub signs in with the same account and *reads those same tables*. There is no syncing, no second save file, no hub copy of anything. If the hub ever gets interactive elements (Phase 3), those actions write to the *same* app tables the apps use — one source of truth, always.

## Architecture

- **Repo name: `brieespo.github.io`** (GitHub user site). The hub renders at `https://brieespo.github.io/` and every app is naturally a path under it (`/dinner-planner/`, `/perfume-tracker/`, `/law-school-tracker/`, `/sewing-tracker/`, `/restock/`). Same-tab navigation — it feels like one site while every app remains a separate repo, separately deployed, other users undisturbed. **Verify each app's actual live path and Supabase table name against reality before wiring its widget** — e.g., the law school app shipped as `law-school-tracker`, not the planned `law-school`.
- **Shared Supabase project** (the dinner planner's). Each app has its own table (`user_data`, `perfume_data`, `law_school_data`, `sewing_data`, `restock_data`), each with row-level security. Adding tables never affects the dinner planner's other users. The hub uses the same email sign-in; one login session per browser covers everything on the supabase side.
- **Stack:** same conventions as all the apps — one file (`hub.html`, `index.html` copy), vanilla JS, Supabase client only, GitHub Pages + same Actions workflow.
- **Visual style:** Law School Command Center language — headline banner, stacked color-coded panels, computed numbers prominent. Mobile stacks vertically.
- **Design language (suite-wide):** no emoji in UI chrome — inline Lucide-style SVG icons (`stroke="currentColor"`, pasted inline, no CDN); CSS dots/chips for statuses; each app's registry icon is the one decorative glyph in its panel header; warmth via accent colors and micro-copy, not decoration.
- **Model escalation:** if a task exceeds your ability (two failed fixes, architectural uncertainty, risky data changes), say so and recommend rerunning on `/model fable`.

## The app registry (extensibility — Bri's requirement)

**Identity now lives in `apps.json`** at the hub root — the single source of truth for the suite's app list, so adding an app is one edit there rather than one per app that lists it. It holds identity only (id, name, url, table, colour vars, icon key); behaviour (headline/panel renderers) and the icon SVG stay in each consumer's JS keyed by id, since neither survives JSON. Sibling apps read it too (the agenda's apps menu) — same origin, so a plain fetch with no CORS or auth, working in guest mode. Every consumer keeps a hand-synced `*_FALLBACK` copy used only when the fetch fails: offline, a bad deploy, or `file://` should leave a stale list, never an empty one.

Adding a future app to the hub must be a config entry + one small function, never a rebuild. Structure the hub around an `APPS` registry:

```js
const APPS = [
  {
    id: 'law',
    name: 'Law School',
    url: '/law-school-tracker/',
    table: 'law_school_data',   // verify actual table name in the law-school-tracker repo
    color: 'var(--law)',        // each app gets a hub accent color
    icon: '⚖️',
    headline: lawHeadline,       // (data) => "lit review due in 6 days" | null
    renderPanel: lawPanel        // (data, el) => fills the panel
  },
  // dinner, perfume, sewing, restock...
];
```

- On load: fetch the signed-in user's row from each registered table (in parallel), then render panels in registry order.
- **Graceful degradation:** table missing, row empty, or fetch error → the panel renders as a simple styled link card to the app. A brand-new app is "on the dashboard" the moment it has a registry entry, even before its panel logic is written.
- Each `headline` function may contribute one line to the top banner; the banner concatenates non-null contributions ("Evidence at 10:20 · dish soap ~Aug 20 · dinner: chicken piccata").
- Panel functions are read-only renderers over the app's jsonb — typically 20–50 lines each.

**Adding a new app end-to-end:** build the app from APP_TEMPLATE.md (its own repo + table) → add one `APPS` entry with `renderPanel: null` (link card appears immediately) → write the panel function when ready. Document this checklist in the hub README.

## Interaction model (decided with Bri)

- **Widgets, not tabs.** The hub is a glanceable dashboard of read-only widgets; apps are never embedded (no iframes). Clicking anywhere on a widget opens that app in the **same tab** — everything lives under brieespo.github.io, so navigation feels like one site and back returns to the dashboard.
- **App-switcher icon row** at the top (one icon per registered app, accent-colored): instant navigation without embedding. This is the "tabs" experience done safely.
- Desktop: 2-column widget grid; mobile: single stacked column, most time-sensitive first.
- Widgets may include **deep-link affordances** where cheap (e.g., tapping the Law Review widget's checkpoint opens the law app's Note Tracker screen via a URL hash the app already supports — only where the app exposes one; never build app features for the hub's sake).

## Widget lineup (v1 — Bri's spec)

- **This Week's Menu (dinner planner):** the 7-day strip from the current week plan, tonight highlighted. Read from the dinner planner's `user_data` table — verify the live shape of the current-plan/saved-menus structure before coding.
- **Upcoming Restocks (restock):** top 3 radar items with urgency dots + "covered until" notes for stocked-deep staples; total attention count feeds the headline banner.
- **Sewing Bench (sewing):** current in-progress project, else top-of-queue, with its readiness line ("just needs a zipper") and target-date warning if close.
- **Law School Deadlines (law school):** next 2–3 items — nearest assignment/exam/milestone lead task, plus today's study blocks when finals mode is active.
- **Law Review Focus (law school table, separate widget):** the note-writing program's current monthly stage, its guiding question ("What conversation do I want to join?"), and days until the next checkpoint. Distinct widget from deadlines — this is orientation, not urgency.
- **Perfume (later, low priority):** scent profile line + samples awaiting review count.

Each widget = one registry entry's `renderPanel` + optional `headline` contribution. The headline banner assembles from whatever widgets report ("Evidence at 10:20 · dish soap ~Aug 20 · dinner: chicken piccata").

## Mixed visual skins across apps — accepted

Three apps look like the dinner planner, two like the Law School Command Center. This is fine: all five share the underlying principles (single file, CSS-variable theming, cards/panels). If Bri later wants visual convergence, the path is cheap: pick the preferred variable palette and apply it app-by-app — a variables-only change, no restructuring. Optional future nicety: each app adds a small hub icon/link in its header (one-line change, made opportunistically, never urgent).

## Build phases

1. **Phase 1:** shell — auth, headline banner scaffold, registry with all five apps as link cards, Law School Command Center styling.
2. **Phase 2:** live read-only panels + headline contributions, app by app (start with whichever apps have real data).
3. **Phase 3:** quick actions that write to app tables (check off a study block, "Bought it") — same tables, same shapes the apps expect; test against each app after.
4. **Phase 4:** polish — panel reordering in settings, per-app accent theming, weekly digest view.

## Open items

1. Confirm all newer apps were in fact created in the shared Supabase project (any that weren't: migrating a jsonb row is a one-time copy).
2. Dinner planner's current-week-plan storage shape — inspect before writing its panel (its CLAUDE.md documents `user_data` columns).
3. Does the existing `dinner-planner` repo's GitHub Pages setup conflict with creating a user site? (It shouldn't — project sites and user sites coexist — but verify the URL paths after the hub deploys.)
