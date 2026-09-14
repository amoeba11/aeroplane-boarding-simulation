# Boarding Call

A single-page simulator that races airplane boarding strategies seat by seat, on any of ten real aircraft. Pick up to 3 boarding orders from the sidebar and they board the same plane side by side, in one row:

- **Case 1 — Window → Middle → Aisle**: everyone with a window seat boards first, then middle, then aisle. Structurally has zero seat conflicts.
- **Case 2 — Back → Middle → Front**: the plane boards in thirds, back section first.
- **Case 3 — Back → Middle-Back → Middle → Middle-Front → Front**: the same back-to-front idea sliced into five zones instead of three.
- **Case 4 — Front → Middle → Back**: the naive reverse of Case 2 — boards the cabin nose-first, one of the slowest real-world approaches.
- **Case 5 — Random (no order)**: passengers board in no particular order at all — a common research baseline.
- **Case 6 — Steffen Method (alternating)**: Jason Steffen's method — window/middle/aisle like Case 1, but alternating aisle side and row parity within each so consecutive boarders are never stowing bags near one another. The fastest known method in boarding research.

Every selected case boards the exact same set of passengers (identical random luggage-stowing times per seat) on a model of the chosen aircraft's real layout — including twin-aisle widebodies, where the sim runs two independent aisle lanes fed by a single boarding queue. A "seat conflict" is counted whenever someone already seated has to stand up so a later passenger can reach a seat further from the aisle — which is why Window → Middle → Aisle and the Steffen method usually win.

## Run it

Open [`index.html`](index.html) in any browser. No build step, no dependencies.

## Aircraft included

Bombardier Q400, Embraer E190, Airbus A220-300, Airbus A320, Boeing 737 MAX 8, Concorde, Boeing 787-9 Dreamliner, Airbus A350-900, Boeing 777-300ER, Airbus A380 (main deck).

## Adding a new boarding order or aircraft

See [CLAUDE.md](CLAUDE.md) for the data model and how to extend either list.
