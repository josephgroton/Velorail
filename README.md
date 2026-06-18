# Velorail

A multimodal trip planner for the Philadelphia region that treats the bicycle as a
first-class transit leg — because the fastest way across this city is usually a bike
*and* a train, and no mainstream app routes that way honestly.

**[Live demo →](https://YOURUSERNAME.github.io/spoke-and-rail/)** *(update this link after enabling Pages)*

## What it does

Set a start and end point (or pick a preset trip) and compare four itineraries side by side:

| Scenario | What it models |
|---|---|
| Walk + transit | The answer Google Maps gives you |
| Bike carried aboard | Your bike rides the train — including real carry rules (PATCO: 24/7. SEPTA: off-peak only) |
| Bike to station | Ride the first mile, lock up, walk the last |
| Indego bikeshare | Dock-to-dock legs, priced by your plan |

Every itinerary shows total time **and marginal cost** under your rider profile —
Indego members and pay-per-ride users see different numbers, SEPTA Metro's free
transfers are modeled, and PATCO fares are exact.

Example: Mt. Airy → Collingswood, NJ. Conventional routing: **88 min, $11.25** across
three fares. Bike carried aboard: **75 min, $5.50.** That gap is the whole product.

## Status

Working prototype. Single self-contained `index.html` — Leaflet map, hand-built network
(MFL, BSL, Chestnut Hill West, full PATCO, sample Indego docks), time-based Dijkstra
router with bike-possession state, average-headway schedules. No backend, no build step.

Known simplifications: headways instead of real GTFS timetables, straight-line × 1.3
distances instead of street routing, Regional Rail fares estimated. See `CLAUDE.md`
for the full architecture notes and roadmap (next up: cost-aware routing where your
time-vs-money preference changes the route itself).

## Running it

It's one file. Open `index.html` in a browser, or serve the repo with GitHub Pages.
