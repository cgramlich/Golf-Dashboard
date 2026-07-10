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
- [~] **Phase 3 — server-side AI + authed sync.**
  - [x] **Authed sync (v1.12.0).** `runSync` now pulls/pushes the four
    collections via `GET/PUT /api/collection/{name}` with the Supabase bearer
    token, gated on being signed in. The GitHub path is demoted to a one-time
    `githubImportOnce` (reachable from the old Sync modal) so existing data can
    be pulled off `data.json` into local state, from where it pushes up.
  - [x] **AI relay (v1.13.0).** The three browser Anthropic calls
    (`callAnthropic`, `callAnthropicChat`, `callAnthropicVision`) now route
    through `aiRelay()` → `/api/ai/relay` with the Supabase token; the
    Anthropic key box is removed and AI is gated on being signed in. Backend
    `ALLOWED_MODELS`/`DEFAULT_MODEL` bumped to `claude-sonnet-5` (backend
    v0.2.0). AI cost now runs on the server key. The AI Settings modal is a
    model picker only.
- [ ] **Phase 4 — data migration.** No script needed: on the primary device
  (which already holds the full log locally), sign in → `runSync` sees an empty
  backend, merges (local wins), and pushes everything up. Do one final GitHub
  import first to be sure local is complete, then retire the `golf-data` path.
- [ ] **Phase 5 — on-course GPS rangefinder.** Precise distances while playing:
  distance to the **center** (and front/back) of the green, and carry distances
  to hazards (bunkers, water), plus "which hole am I on." This needs surveyed
  green + hazard coordinates, which our scorecard API does **not** have.

## Data providers (decided)

Layered, with a clear fallback order — the app should degrade gracefully, never
dead-end:

- **Green / hazard / hole GPS coordinates → paid provider (primary).** Trialing
  **Golf Intelligence** (green centers + hazards + tee boxes, rangefinder-focused,
  free 200-credit test tier); **golfapi.io** is the value alternative that also
  bundles scorecards. Chosen on which one actually has *our* courses mapped.
- **Scorecards (par / stroke index / yardage) → paid provider first, then
  GolfCourseAPI as fallback.** If the paid GPS provider also serves the scorecard
  (golfapi.io / Golf Intelligence do), use it so hole numbers line up with the GPS
  data; if it's missing a course's card, fall back to **GolfCourseAPI** (kept, free).
- **Nearby course list + rough hole detection → OpenStreetMap / Overpass (free).**
  Unchanged. The paid provider can later supersede "detect hole" once its data is
  in place.
- **Cost control:** fetch a course's geometry **once, cache it in Supabase**, and
  compute live distances server/client-side from the stored coords + phone GPS —
  so the paid API is hit ~once per course, not per shot. Keeps spend at pennies for
  a single-user app and argues for optimizing on data quality, not price.

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
