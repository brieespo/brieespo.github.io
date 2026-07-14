# Hub

The dashboard that sits above Bri's suite of personal apps — [dinner-planner](https://github.com/brieespo/dinner-planner), [law-school-tracker](https://github.com/brieespo/law-school-tracker), [sewing-tracker](https://github.com/brieespo/sewing-tracker), [restock](https://github.com/brieespo/restock), [perfume-tracker](https://github.com/brieespo/perfume-tracker). Pure HTML + CSS + vanilla JS in a single file, Supabase for auth, GitHub Pages for hosting.

**Live at:** https://brieespo.github.io/

## The one rule that makes this work

The hub owns no app data. Every app writes to its own table in the shared Supabase project; the hub signs in with the same account and reads those same tables read-only. No syncing, no second save file, no hub copy of anything.

## Files

- `hub.html` — the entire app (source of truth)
- `index.html` — always an exact copy of `hub.html`. After every change: `cp hub.html index.html`, then push.
- `.github/workflows/deploy.yml` — GitHub Pages deploy on every push to `main`

## Adding a new app to the registry

1. Build the app from its own `APP_TEMPLATE.md` (own repo + own Supabase table).
2. Add one entry to the `APPS` array in `hub.html` with `url`, `table`, `color`/`bg` CSS vars, and `icon` — omit `panels` and it shows up immediately as a link card.
3. When ready, add a `panels` array: `{ title, render(data, bodyEl) }` for each widget, and optionally a `headline(data)` function that contributes one line to the top banner.
4. `cp hub.html index.html` and push.

## Supabase

Reuses the shared Supabase project (same URL and publishable key as every sibling app). No new table — the hub only reads `law_school_data`, `user_data` (dinner planner), `sewing_data`, `restock_data`, and `perfume_data`, one row per user via `.eq('user_id', user.id).maybeSingle()`.

## Current widget lineup

- **Law School Deadlines** + **Law Review Focus** — nearest milestone/lead-task deadlines, and the Volume 65 note-writing roadmap's current stage (a fixed schedule, not per-user data).
- **This Week's Menu** — the most recently saved weekly menu (the dinner planner doesn't sync its live in-progress plan, only saved menus, so this is the closest available proxy for "current week").
- **Sewing Bench** — the in-progress project, else top of the queue.
- **Restock** and **Perfume** — link cards for now; add `panels` once their widgets are written.
