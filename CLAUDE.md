# Spoke & Rail — project brief

## What this is
A multimodal trip planner that treats the bicycle as a first-class transit leg. It compares,
side by side, four ways to make the same trip across Philadelphia-region transit (SEPTA + PATCO):

1. **Walk + transit** — the baseline every mainstream app gives you ("the Google answer")
2. **Bike carried aboard** — personal bike rides the train with you
3. **Bike to station, lock it** — bike the first mile, walk the last
4. **Indego bikeshare** — dock-to-dock bike legs (Philadelphia side only; Indego has no Camden docks)

## Product thesis (why this beats Citymapper/Transit/Google)
The differentiator is NOT "bike+transit routing exists" — Citymapper and Transit+ both do
crude versions. The differentiator is **profile-driven routing with honest cost modeling**:

- The user declares a profile: owns a bike (y/n), willing to carry it aboard (y/n),
  Indego plan (none / pay-per-ride / member).
- The router knows the **real carry-aboard rules**: PATCO allows bikes 24/7; SEPTA
  (Metro + Regional Rail) bars bikes on board during weekday peak (6–9a, 3–6p).
  Mainstream apps ignore this entirely.
- Every itinerary shows **marginal cost** under the user's profile. An Indego member's
  bike leg costs $0; a casual rider's costs $4.50. SEPTA Metro's two free transfers
  within 120 min are modeled (one $2.90 covers MFL+BSL). This changes which route is best.
- Demo result that sells the thesis: Mt. Airy → Collingswood is 88 min / $11.25 the
  Google way (Regional Rail + Metro + PATCO, three fares) but 75 min / $5.50 with a
  bike carried aboard, because biking to the BSL skips the Regional Rail fare entirely.

## Current state: index.html (self-contained prototype)
Single-file web app. Leaflet (cdnjs) + CARTO light tiles. No build step, no backend.
Deployable as-is to GitHub Pages.

### Architecture inside the file
- **Data**: `LINES`, `STATIONS`, `SEQ` (track order per line), `SEGMIN` (minutes between
  adjacent stations), `TRANSFERS` (explicit walking transfers), `DOCKS` (sample Indego),
  `PRESETS` (demo trips). Network covers: Market–Frankford Line, Broad Street Line,
  Chestnut Hill West Regional Rail, full PATCO line, ~13 Indego docks. Coordinates approximate.
- **Routing engine**: time-based Dijkstra over a state graph. Node keys: `O` (origin),
  `D` (destination), `S:<id>` (station street level), `P:<id>:<line>` (platform — entering
  a platform costs headway/2 + 1 min wait), `K:<id>` (Indego dock). State = nodeKey + `#` +
  bike-possession bit. Scenario rules live in `neighbors()`:
  - carry: platform entry with bike requires `lineAllowsBike(line, peak)`
  - lock: a lock edge (1.5 min) drops the bike bit before boarding
  - indego: bike edges only dock-to-dock (+1.5 min dock/undock)
- **Leg merging** (`mergeLegs`): consecutive rides on a line merge; waits fold into the
  following ride; consecutive same-mode walk/bike legs merge with summed distance.
- **Fare engine** (`tripCost`): PATCO exact point-to-point fares by station group;
  SEPTA Metro $2.90 charged once per itinerary (free-transfer rule); Regional Rail
  zone fares ESTIMATED (`RR_ZONE_FARE`) — see data notes; Indego $4.50/30min single
  ride (+$0.30/min overage) or $0 for members.
- **UI**: profile panel drives which scenario cards render; cards are strip-map style
  with per-leg minutes and a cost chip; clicking a card highlights its polyline.

### Speeds & assumptions
Bike 11 mph, walk 3.1 mph, straight-line distance × 1.3 street circuity factor.
Walk access cap 1.05 mi, bike access cap 6.5 mi. Headways: MFL 6, BSL 8, PATCO 12,
CHW 35 min (average-headway model, NOT real timetables).

## Verified data (as of June 2026 — re-verify before launch)
- SEPTA Bus/Metro base fare: **$2.90**, two free transfers within 120 min (contactless/Key).
  Source: septa.org FY26 budget release + fares FAQ.
- SEPTA Regional Rail: zone-based, no free transfers, up to $13 max. The Z1=$5.25 /
  Z2=$5.75 figures in the code are ESTIMATES — replace with the official zone table.
  Note: Tulpehocken became Zone 1 in Dec 2024.
- PATCO fares (exact, from ridepatco.org timetable): to/from Philly — Lindenwold/Ashland/
  Woodcrest $3.00; Haddonfield/Westmont/Collingswood $2.60; Ferry Ave $2.25;
  Broadway/City Hall (Camden) $1.40; NJ↔NJ $1.60; within Philly $1.40.
- Indego (rideindego.com): Single Ride $4.50/30 min classic, +$0.30/min after;
  Indego30 $20/mo; Indego365 $156/yr (hour-long classic rides included);
  e-bike surcharge $0.20/min for members.
- Bike rules: PATCO allows bikes all hours. SEPTA allows bikes on Metro and Regional
  Rail off-peak only (weekday peak ≈ 6–9a, 3–6p). Verify exact current SEPTA policy text.

## Roadmap
1. **v3 — cost-aware routing (next)**: optimize a blended objective
   `generalized_cost = minutes + λ × dollars`, with λ exposed as a profile slider
   ("I'd ride N extra minutes to save $1"). This makes a member and a casual rider get
   genuinely different ROUTES, not just different price tags. Implementation: add fare
   increments to edge weights in Dijkstra (board edges carry the fare; metro free-transfer
   needs a small fare-state bit or post-hoc correction).
2. **Arrive-by / schedule planning**: Let users input a target arrival time. The app
   back-calculates departure windows and flags whether the trip is feasible with/without
   a bike under current SEPTA/MBTA timetables. This is a planning tool, not real-time —
   compute against published schedules, not live departures. GTFS feeds required.
3. **Better LTS street weighting**: Route bike legs along low-stress streets (LTS 1–2)
   even when slightly slower. Use OSM cycleway/lane/surface tags + LTS classification.
   Mainstream apps route bikes like cars; this is the core differentiator on the street level.
4. **More network**: key bus routes, MFL/BSL trolleys, NJ Transit River Line, real Indego
   dock list from the GBFS feed (https://gbfs.bcycle.com/bcycle_indego/gbfs.json — verify URL).
   Boston: add Bluebikes, Silver Line, Commuter Rail.
5. **Production path**: OpenTripPlanner 2 self-hosted with real GTFS (SEPTA, NJ Transit,
   PATCO publishes GTFS — see ridepatco.org "Developers (GTFS)") + OSM extract.
   OTP handles bike+transit and bikeshare natively; the custom work is the cost model
   (per-line bike-aboard penalties by time of day, bike-stress street weighting) and this UI.
   Target: VPS with 8–16 GB RAM (~$40–80/mo).

## Conventions
- Keep the prototype a single self-contained index.html until OTP migration — it must
  stay deployable on GitHub Pages with zero build.
- Use only cdnjs.cloudflare.com for libraries.
- All fares carry a source comment. Estimated figures are marked ESTIMATED in code and
  shown with "~" in the UI.
- Test the engine headlessly: extract the two <script> blocks, concatenate, run preset
  trips through `route()` in Node and sanity-check times/fares before shipping UI changes.

## Owner context
Joey — construction project manager, elite endurance cyclist, Philadelphia (planning a
move to Mt. Airy). Knows SEPTA/PATCO as a daily-life user, comfortable with spreadsheets
and data models, new to software deployment. Explain infra decisions plainly; don't
assume web-dev background.
