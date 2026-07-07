# LinksCaptain — Roadmap & Cleanup

App brand: **LinksCaptain** (a **Salty Tee Box** product). Studio + app model,
mirroring the Menu Captain family. The pirate-skull logo and green/gold palette
are the house identity and stay.

Live at **https://linkscaptain.com** (GitHub Pages + custom domain, HTTPS on).

## Repos

- **`LinksCaptain-App`** — this repo, the front-end PWA (renamed from `Golf-Dashboard`).
- **`LinksCaptain-Cloud`** — the backend API (future; does not exist yet).
  FastAPI + Supabase + server-side Anthropic. Successor to the `golf-data` sync.
- **`golf-data`** — today's sync store (one `data.json`). **Not renamed** — it is
  slated to be *retired* once data moves into Supabase (see Phase 4), so renaming
  it would be churn and would force re-pointing the in-app sync config per device.

## Architecture direction

Aligning with the Menu Captain (`dining-log-app` + `dining-captain-backend`)
production pattern, minus commerce/store for now:

- **Front-end:** installable PWA (`LinksCaptain-App`), served from the custom domain.
- **Backend (future):** `LinksCaptain-Cloud` — FastAPI + Supabase (Postgres + JWT
  auth), server-side Anthropic relay. Its own separate infra (new Supabase project
  + hosting), not shared with Menu Captain.
- **Stripe / app stores:** structure stubbed/disabled for now (entitlement layer
  hardwired to unlimited, à la Menu Captain's `/api/entitlement` "pro" stub), no
  store-listing pages yet.

## Phases

- [x] **Phase 1 — PWA shell parity.** manifest.json, service worker (sw.js),
  real icon set. Keeps the existing GitHub `data.json` sync.
- [x] **Rebrand to LinksCaptain.** All in-app brand touchpoints + manifest.
- [x] **Custom domain.** `linkscaptain.com` on GitHub Pages (DNS at Porkbun,
  `CNAME` file, Enforce HTTPS). Confirmed installable over HTTPS.
- [x] **Rename front-end repo** `Golf-Dashboard` → `LinksCaptain-App`. Domain
  insulated the URL; no functional code changes were needed.
- [ ] **Phase 2 — backend.** Stand up `LinksCaptain-Cloud` (FastAPI + Supabase),
  golf schema (entries/courses/rounds/analyses), entitlement stub. **Needs infra
  first:** a new Supabase project + a hosting account (Railway or Render).
- [ ] **Phase 3 — server-side AI + authed sync.** Move Anthropic calls to a
  `/api/ai/relay`; switch sync from GitHub PAT → Supabase-JWT API calls.
- [ ] **Phase 4 — data migration.** Move existing `data.json` into Supabase,
  then retire the `golf-data` GitHub-token sync path.

## Potential upgrades (parked)

- **Paid / self-hosted Overpass for "Near me".** Today the nearby-courses search
  races several free public Overpass mirrors (volunteer copies of the OSM query
  service). The fix in place (prefer a mirror that actually returns courses,
  ignore fast-but-empty answers) makes this reliable, but the mirrors are still
  third-party and best-effort. If they get flaky, the clean escape hatch is a
  **paid Overpass/geo provider** (a few $/mo, guaranteed uptime + complete data,
  no mirror roulette) or **self-hosting** our own Overpass/PostGIS instance
  (full control, but real infra: ~150 GB planet dump, a bigger server, and
  keeping it updated). Paid provider is the pragmatic middle option.

## Notes / open items

- **Home-screen cutover:** re-add the app to the phone home screen from
  `linkscaptain.com` so the installed PWA lives on the stable domain (and picks
  up the new LinksCaptain icon). The old `*.github.io/LinksCaptain-App/` URL keeps
  working via GitHub's redirect.
- **Default AI model** in the app is `claude-sonnet-4-6` — worth confirming it is
  still a current model id (Opus/Haiku ids looked current).
