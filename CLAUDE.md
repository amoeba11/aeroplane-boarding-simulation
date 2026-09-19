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

## Deployment

This repo is also served as a static site via GitHub Pages, from `main` / root —
`index.html` is what's live at https://amoeba11.github.io/aeroplane-boarding-simulation/.
Any push to `main` deploys automatically (a few seconds to a minute to rebuild);
there's no separate deploy step to remember.

The masthead's "VISITS" field is a third-party hit-counter badge
(`hits.sh/amoeba11.github.io/aeroplane-boarding-simulation.svg`) — an `<img>`,
not a script, so it needs no dependency. It has an `onerror` fallback that hides
the whole field if the badge fails to load, which is also what happens when this
same `index.html` is opened as a Claude Artifact instead of via GitHub Pages: the
Artifact sandbox blocks `<img>` tags from arbitrary external hosts, so the badge
silently fails there and the field just doesn't show — expected, not a bug.

## Core data model

Two arrays drive everything and are the main extension points:

- **`AIRCRAFT`** — one entry per real aircraft: `{ id, name, tagline, blocks, rows }`.
  `blocks` is the seat layout as block sizes left to right, e.g. `[3,3]` for a
  standard 3-3 narrowbody, `[3,4,3]` for a 10-abreast widebody (two aisles).
  Do not hand-assign seat letters or aisle positions — `buildModel()` derives all
  of that from `blocks`.
- **`STRATEGIES`** — one entry per boarding order shown in the left sidebar:
  `{ num, id, kind, label, accent, zones? }`. `num` is a fixed, permanent identity
  ("Case N") — it must never be reassigned or renumbered based on which cases are
  currently selected or how many exist; selection only controls visibility.
  `accent` picks a categorical color slot (`c1`..`c6`, defined as CSS custom
  properties, one per strategy currently defined). Assign accents in a fixed
  order as strategies are added — never recolor an existing case when a new one
  is added. `kind` selects which builder function `buildForStrategy()` dispatches
  to (see below).

Users may select **at most `MAX_SELECTED` (currently 3)** strategies to race at
once — enforced in `updateCaseAvailability()`, which disables the remaining
unchecked checkboxes once the cap is hit. If you raise the cap, also reconsider
`.planes`'s `grid-template-columns: repeat(var(--panels,1), 1fr)` (set from
`runners.length` in `setup()`), which lays every active case out in a single
horizontal row on desktop — more than 3–4 will get cramped and may need a wrap
strategy instead of forcing one row.

### Adding a new aircraft

Append to `AIRCRAFT` with real, publicly known seating figures (rows × layout).
Keep `blocks` accurate to how the real aircraft is configured (single-aisle
`[a,b]` vs twin-aisle `[a,b,c]`) — the simulation's realism (aisle contention,
seat-climbing interference) depends on this being right, not just the seat count.

### Adding a new boarding order ("Case")

1. Add an entry to `STRATEGIES` with the next `num` and an unused accent slot
   (add a new `--cN`/`--cN-ink` pair in all three theme places — bare `:root`,
   the `prefers-color-scheme: dark` block, and `:root[data-theme="dark"]` —
   copy an existing pair as a template; no per-property `.panel[data-accent=...]`
   rules are needed since those read `var(--accent)`/`var(--accent-ink)`, set
   once per panel from its `data-accent`).
2. Give it a `kind` and extend `buildForStrategy()`'s switch if it's a genuinely
   new ordering approach, or reuse an existing `kind`:
   - `'zones'` / `'zonesReverse'` — row-based sections, back-to-front or
     front-to-back. Set `zones: N`; if `N` isn't already in `ZONE_NAMES`, add a
     front-to-back name list there (used to label sections like "Middle-Back
     Section · Rows 19–24"). `buildZones()` takes a `reverse` flag rather than
     having two code paths.
   - `'wma'` — seat-type grouping (window/middle/aisle), zero structural seat
     conflicts by construction.
   - `'random'` — one unordered group, a research baseline.
   - `'steffen'` — Jason Steffen's alternating method (window→middle→aisle, but
     each split further by aisle-side and row parity into 4 interleaved passes
     so consecutive boarders are never stowing bags near each other). Uses the
     per-seat `side` field.

Every strategy must return `{ order, groupSizes, groupLabels }` where `order` is
a flat array of seat IDs (boarding sequence) and `groupSizes`/`groupLabels` describe
the phases shown as chips and in the "Now boarding: …" line.

## How the simulation works

- Seats are `{row}{letter}` IDs (e.g. `"14A"`), letters skip `I` per aviation
  convention (`LETTERS` constant).
- Each seat is pre-classified with an `aisleIdx` (which aisle lane it boards
  through — relevant for twin-aisle aircraft), a `side` (`'left'`/`'right'` —
  which side of that aisle it approaches from, used by the Steffen method), and
  `neighborIds` (the seats between it and its aisle, used to detect interference).
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
