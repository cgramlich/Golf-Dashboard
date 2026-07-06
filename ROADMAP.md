# LinksCaptain — Roadmap & Cleanup

App brand: **LinksCaptain** (a **Salty Tee Box** product). Studio + app model,
mirroring the Menu Captain family. The pirate-skull logo and green/gold palette
are the house identity and stay.

The GitHub repos keep their current names for now (`Golf-Dashboard`,
`golf-data`, and the future `golf-backend`). Renaming is deliberately deferred —
see **Cleanup** below.

## Architecture direction

Aligning with the Menu Captain (`dining-log-app` + `dining-captain-backend`)
production pattern, minus commerce/store for now:

- **Front-end:** installable PWA (this repo), served from GitHub Pages today,
  a custom domain later.
- **Backend (future):** new **`golf-backend`** repo — FastAPI + Supabase
  (Postgres + JWT auth), server-side Anthropic relay. Its own separate infra
  (new Supabase project + hosting), not shared with Menu Captain.
- **Stripe / app stores:** structure stubbed/disabled for now (entitlement layer
  hardwired to unlimited), no store-listing pages yet.

## Phases

- [x] **Phase 1 — PWA shell parity.** manifest.json, service worker (sw.js),
  real icon set. Keeps the existing GitHub `data.json` sync.
- [x] **Rebrand to LinksCaptain.** All in-app brand touchpoints + manifest.
  Repos intentionally untouched.
- [ ] **Custom domain.** Point a domain (e.g. `linkscaptain.com` or a subdomain)
  at the app via a `CNAME` file + DNS. **This must land and be verified working
  before any repo rename** — a stable domain insulates the installed home-screen
  PWA from repo-name-driven URL changes.
- [ ] **Phase 2 — backend.** Stand up `golf-backend` (FastAPI + Supabase),
  golf schema (entries/courses/rounds/analyses), entitlement stub.
- [ ] **Phase 3 — server-side AI + authed sync.** Move Anthropic calls to a
  `/api/ai/relay`; switch sync from GitHub PAT → Supabase-JWT API calls.
- [ ] **Phase 4 — data migration.** Move existing `data.json` into Supabase,
  retire the GitHub-token sync path.

## Cleanup (deferred — do AFTER the custom domain is live and verified)

**Repo renames.** Once the domain fronts the app so the public URL is stable:

- `Golf-Dashboard` → new name (TBD, e.g. `linkscaptain` / `linkscaptain-app`)
- `golf-data` → matching data-repo name
- keep `golf-backend` naming consistent with the above

Steps when the time comes:

1. Confirm the custom domain serves the app and the installed PWA loads from it
   (not from `*.github.io/Golf-Dashboard/`).
2. **Manual, one-time:** rename each repo in GitHub → Settings → Rename. This is
   the only step Claude cannot do — the GitHub connector exposes no repo-rename
   tool, and it is repo-admin gated.
3. Claude does the rest unattended: update every code/config reference to the old
   names (in-app sync config default, backend repo references, any hardcoded
   URLs, docs), reconfirm the `CNAME`, and re-verify the app end-to-end, then
   commit + push.

> Why deferred: renaming a repo now would change the GitHub Pages project URL
> (`/Golf-Dashboard/` → `/NewName/`), which GitHub does not reliably redirect,
> breaking the installed home-screen app and the `data.json` sync until
> reconfigured. With a custom domain in place first, the rename is free.
