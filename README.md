# Boarding Call

A single-page simulator that races airplane boarding strategies seat by seat, on any of ten real aircraft. Pick any combination of boarding orders from the sidebar and they all board the same plane at once:

- **Case 1 — Window → Middle → Aisle**: everyone with a window seat boards first, then middle, then aisle.
- **Case 2 — Back → Middle → Front**: the plane boards in thirds, back section first.
- **Case 3 — Back → Middle-Back → Middle → Middle-Front → Front**: the same back-to-front idea sliced into five zones instead of three.

Every selected case boards the exact same set of passengers (identical random luggage-stowing times per seat) on a model of the chosen aircraft's real layout — including twin-aisle widebodies, where the sim runs two independent aisle lanes fed by a single boarding queue. A "seat conflict" is counted whenever someone already seated has to stand up so a later passenger can reach a seat further from the aisle — which is why Window → Middle → Aisle usually wins.

## Run it

Open [`index.html`](index.html) in any browser. No build step, no dependencies.

## Aircraft included

Bombardier Q400, Embraer E190, Airbus A220-300, Airbus A320, Boeing 737 MAX 8, Concorde, Boeing 787-9 Dreamliner, Airbus A350-900, Boeing 777-300ER, Airbus A380 (main deck).

## Adding a new boarding order or aircraft

See [CLAUDE.md](CLAUDE.md) for the data model and how to extend either list.
