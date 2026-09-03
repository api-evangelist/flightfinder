---
name: flightfinder-aircraft-safety-record
description: Build a defensible safety record for one aircraft family from FlightFinder's merged multi-authority corpus, with per-source attribution and licence-correct narrative handling.
api: FlightFinder Aviation Safety Data API
base_url: https://himaxym.com/api/v1/data
generated: '2026-09-03'
method: generated
source: openapi/flightfinder-aviation-safety-data-openapi.json (the served spec declares no operationId; operations are addressed as METHOD + path)
operations:
  - GET /aircraft/{family}/safety
  - GET /events
  - GET /events/{id}
  - GET /narratives/{source}/{id}
  - GET /sources
mcp_tools:
  - aircraft_safety
  - search_accidents
  - get_accident
  - get_narrative
  - list_sources
---

# Aircraft family safety record

Answer "how safe is this aircraft family" with numbers you can cite, not a score you cannot defend.

## Before you start

- **No credential is required.** Every GET below answers with no `Authorization` header at 100 requests/day/IP. Only add `Authorization: Bearer <key>` when you need more than that.
- Family is a **slug** (`boeing-737`, `airbus-a320`, `boeing-787`), not a marketing name.
- If you have an MCP client, the same steps run as `aircraft_safety`, `search_accidents`, `get_accident`, `get_narrative` against `https://himaxym.com/mcp` — also anonymous.

## Steps

1. **Get the aggregate.** `GET /aircraft/{family}/safety` returns total occurrences, hull losses, fatal accidents, total fatalities and first/latest year for the family. This is the headline, and it is the only number you should present as a family-level total.

2. **Pull the events behind it.** `GET /events?family={family}&limit=100` and follow `next_cursor` until it is null. Add `fatal=1` for fatal-only, `from=YYYY-MM-DD`/`to=YYYY-MM-DD` to window it, `country=<ISO 3166-1 alpha-2>` to scope it. Filters combine.

3. **Open the ones that matter.** `GET /events/{id}` returns the occurrence plus `sources[]` (each with `source`, `attribution`, `license`, `url`) and refs to the narratives available for it.

4. **Read a narrative only through its licence.** `GET /narratives/{source}/{id}` where `source` is an authority code from `GET /sources` and `id` is that authority's own case id (e.g. `atsb` / `AO-2024-001`). Check `policy`:
   - `full` — open-licensed source, `narrative_text` is the whole thing.
   - `excerpt` — you get ~300 characters plus `source_url`. **Fetch the rest from `source_url` under that source's own licence. Do not reconstruct it, do not paraphrase it back to full length, do not present the excerpt as the complete narrative.**

5. **Resolve the licence before you publish.** `GET /sources` returns every authority with its `license`, `homepage`, narrative count and policy. Cache it — it changes rarely and it is 32KB.

## Rules you must not break

- **Visible attribution is mandatory.** Every record carries `source`, `attribution`, `license` and a deep link. Anything you display or republish must show visible attribution linking the original source.
- **Never compare a family total to another family without normalising.** The corpus has no fleet-hours denominator. Raw occurrence counts favour rare types. Say so.
- **Do not present the aggregate as a risk prediction.** FlightFinder's own methodology page says a two-point difference between two modern types is noise.
- **Pro-tier `article.full_text` is no-public-republication.** Internal use, applications and research only; open-web republication is forbidden and a visible link to the canonical FlightFinder page is required.

## Failure handling

| Status | code | Do this |
|---|---|---|
| 401 | `bad_key` | The key is unknown or revoked. Drop the header entirely and fall back to the keyless tier, or mint a new key. |
| 401 | `query_key_unsupported` | You put the key in the query string. Move it to `Authorization: Bearer`. |
| 404 | `not_found` | Wrong slug or case id. Re-check the family slug, or list sources first. |
| 429 | `rate_limited` | Read `Retry-After` (seconds) and wait it out. `RateLimit-Remaining` on every response tells you this is coming — throttle before you hit it. |

Errors are `{"error":{"code","message"}}` on all data routes. Full catalogue: `errors/flightfinder-problem-types.yml`.
