---
name: flightfinder-hazard-briefing
description: Produce a wildlife-strike, laser-incident or drone-sighting briefing for a US airport, state or species from FlightFinder's FAA-derived hazard datasets — all US-government public domain.
api: FlightFinder Aviation Safety Data API
base_url: https://himaxym.com/api/v1/data
generated: '2026-09-03'
method: generated
source: openapi/flightfinder-aviation-safety-data-openapi.json (the served spec declares no operationId; operations are addressed as METHOD + path)
operations:
  - GET /wildlife-strikes
  - GET /wildlife-strikes/airport/{slug}
  - GET /wildlife-strikes/species/{slug}
  - GET /laser-strikes
  - GET /laser-strikes/state/{slug}
  - GET /laser-strikes/airport/{slug}
  - GET /drone-sightings
  - GET /drone-sightings/state/{slug}
  - GET /airports/{ident}
mcp_tools:
  - wildlife_strikes
  - laser_incidents
  - drone_sightings
  - airport_reference
---

# FAA hazard briefing

Three FAA datasets, one shape: call the bare endpoint for the national picture, add a facet path segment for one airport, state or species.

## The three surfaces

| Hazard | Aggregate | Facets |
|---|---|---|
| Wildlife strikes | `GET /wildlife-strikes` | `/airport/{slug}`, `/species/{slug}` |
| Laser incidents | `GET /laser-strikes` | `/state/{slug}`, `/airport/{slug}` |
| Drone sightings | `GET /drone-sightings` | `/state/{slug}` |

All three are **US-government public domain**. That is the one part of this corpus you may republish freely — but still attribute the FAA, and still link FlightFinder for the aggregation.

## Steps

1. **Anchor on the national series first.** The bare endpoint returns totals, the full yearly series and the leading facets (airports, species, flight phases for wildlife; states and cities for drones). Never publish a single-airport number without the national denominator next to it.

2. **Resolve the airport before you slug it.** `GET /airports/{ident}` takes an **ICAO ident or IATA code** (`KDEN`, `DEN`) and returns name, city, country and coordinates from OurAirports (CC0). Use it to confirm you have the right field before you build a facet URL.

3. **Facet slugs are lowercase and hyphenated** — `kden`, `mourning-dove`. They are not the same string as the ICAO ident you passed to `/airports/{ident}`; lower-case it.

4. **A 404 is an answer.** `not_found` on a facet means that airport, state or species has no reports in the dataset — report zero, do not report "no data available" as if the query failed.

## Rules you must not break

- **Do not mix the hazard datasets with the accident corpus.** A wildlife strike is not an occurrence in `/events`. They have different provenance, different denominators and different licences.
- **Wildlife strike counts are reports, not incidents.** Reporting is voluntary in parts of the FAA programme; a rising series can be rising reporting. Say which you mean.
- **These endpoints return whole aggregate documents** — they are not paginated and their response bodies are untyped in the spec. Read defensively.

## Failure handling

Same envelope as the rest of the API: `{"error":{"code","message"}}`. `429` carries `Retry-After`; every response carries `RateLimit-Remaining`, so back off before you are cut off. Full catalogue: `errors/flightfinder-problem-types.yml`.
