# CLAUDE.md

Guidance for AI agents (and humans) working in this repo.

## What this is

`index.html` is the entire project: a self-contained HTML/CSS/JS page that simulates
and races airplane boarding strategies, seat by seat, on a chosen real aircraft.
There is no build step, no package.json, no dependencies besides two Google Fonts
(Overpass for display/body text, IBM Plex Mono for data/labels) loaded via
`<link>`. Keep it that way — do not introduce a bundler, framework, or npm
dependency for what is a single static page.

To preview changes, just open `index.html` in a browser. There is no test suite;
verify manually by watching a race run to completion (default speed finishes in
under a minute) and checking the Results panel at the end.

## Core data model

Two arrays drive everything and are the main extension points:

- **`AIRCRAFT`** — one entry per real aircraft: `{ id, name, tagline, blocks, rows }`.
  `blocks` is the seat layout as block sizes left to right, e.g. `[3,3]` for a
  standard 3-3 narrowbody, `[3,4,3]` for a 10-abreast widebody (two aisles).
  Do not hand-assign seat letters or aisle positions — `buildModel()` derives all
  of that from `blocks`.
- **`STRATEGIES`** — one entry per boarding order shown in the left sidebar:
  `{ num, id, label, accent, zones? }`. `num` is a fixed, permanent identity
  ("Case N") — it must never be reassigned or renumbered based on which cases are
  currently selected; selection only controls visibility. `accent` picks a
  categorical color slot (`c1`/`c2`/`c3`, defined as CSS custom properties). Assign
  accents in a fixed order as strategies are added — never recolor an existing
  case when a new one is added.

### Adding a new aircraft

Append to `AIRCRAFT` with real, publicly known seating figures (rows × layout).
Keep `blocks` accurate to how the real aircraft is configured (single-aisle
`[a,b]` vs twin-aisle `[a,b,c]`) — the simulation's realism (aisle contention,
seat-climbing interference) depends on this being right, not just the seat count.

### Adding a new boarding order ("Case")

1. Add an entry to `STRATEGIES` with the next `num` and an unused accent slot
   (add a new `--c4`/`--c4-ink` pair in both the light and dark `:root` blocks if
   you're past `c3`, plus the matching `.panel[data-accent="c4"] ...` CSS rules —
   copy the `c3` block as a template).
2. If it's a row-zone strategy (like Case 2/3), give it a `zones: N` and, if `N`
   isn't already in `ZONE_NAMES`, add a front-to-back name list there (used to
   label sections like "Middle-Back Section · Rows 19–24"). Boarding order for
   zone strategies always goes back-to-front regardless of zone count —
   `buildZones()` reverses the front-to-back list automatically.
3. If it's a seat-type strategy (like Case 1's window/middle/aisle), add a
   `build*()` function following `buildWMA()`'s shape and branch to it in
   `buildForStrategy()`.

Every strategy must return `{ order, groupSizes, groupLabels }` where `order` is
a flat array of seat IDs (boarding sequence) and `groupSizes`/`groupLabels` describe
the phases shown as chips and in the "Now boarding: …" line.

## How the simulation works

- Seats are `{row}{letter}` IDs (e.g. `"14A"`), letters skip `I` per aviation
  convention (`LETTERS` constant).
- Each seat is pre-classified with an `aisleIdx` (which aisle lane it boards
  through — relevant for twin-aisle aircraft) and `neighborIds` (the seats between
  it and its aisle, used to detect interference).
- `step()` advances one tick: passengers move one row toward their seat if the
  next row in their aisle lane is free, then stow (a random 4–8 tick delay) once
  they reach their row. A **seat conflict** is counted whenever a passenger starts
  stowing and one of their `neighborIds` is already seated (someone has to stand
  up to let them by) — this is what makes Window→Middle→Aisle structurally
  conflict-free and back-to-front strategies slower.
- All selected strategies simulate the same aircraft with the same per-seat
  luggage-stow times (`seatMetaShared`, seeded once per race) so comparisons are
  apples-to-apples — only the boarding order differs.
- Rendering is live/incremental (`render()` mutates existing DOM nodes each tick),
  not a precomputed frame buffer. If you need scrubbing/rewind, that would require
  recording per-tick snapshots instead — a deliberate simplification, not an
  oversight.

## Style conventions

- Follow the existing "boarding pass / airport departures board" visual language
  (ticket-stub perforation divider, monospace data readouts, barcode motif) rather
  than introducing a different visual identity.
- All colors are CSS custom properties defined once on bare `:root`, redefined for
  dark mode under both `@media (prefers-color-scheme: dark)` (guarded by
  `:not([data-theme="light"])`) and `:root[data-theme="dark"]`. Any new color must
  follow this same three-place pattern or it will only work in one theme.
- Cell size is controlled by `--cell` (and its mobile override) — don't hardcode
  pixel sizes for seats/tokens elsewhere; everything derives from `--cell`,
  `--gap`, `--rowh`, `--headh`.
