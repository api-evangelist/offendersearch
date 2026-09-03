---
generated: '2026-09-03'
method: generated
name: Screen a person against all 58 US sex-offender registries
description: One synchronous call that searches every US registry, then read the completeness labels before trusting an empty result.
api: openapi/offendersearch-openapi.json
operations: [syncSearch, getRecord]
source: >-
  Grounded in openapi/offendersearch-openapi.json (operationIds verified) and
  https://offendersearch.app/docs/search.md.
---

# Screen a person

Search all 58 US registries (or a named subset) in one round trip and get scored, de-duplicated, source-cited records.

## Auth
- Send your secret key in the `X-API-Key` header (`os_live_...`). See `authentication/offendersearch-authentication.yml`.

## Steps
1. **Search** — `syncSearch` (`POST /v1/search`) with a `query` object: `{"query": {"firstName": "John", "lastName": "Doe", "dob": "1980-04-12"}}`. A name + DOB query is never refused; a `firstName` alone returns 422 (deliberately declined, show `error.message` to the user).
2. **Check completeness before reading records** — a 200 can be partial. Read `status`, `counts.sourcesComplete` vs `counts.sourcesQueried`, and `sourceStatus[].incompleteReason` (closed enum: deadline, truncated, excluded, not_searched, unavailable, error). An empty `records` array is only "not found" when `counts.sourcesIncomplete == 0`.
3. **Read the matches** — each record carries `matchConfidence`, `matchBasis`, and per-source provenance in `sources[]` (registry name, record URL, `lastCheckedAt`) the agent can cite.
4. **Fetch one record later if needed** — `getRecord` (`GET /v1/records/{recordId}`). NOTE: `recordId` describes the merge winner for YOUR search scope and is not a stable person key.

## Options
- Bound latency with `deadlineMs` (default partial results; `onDeadline: "error"` returns 504 instead).
- Re-verify at the source in-request with a `live` block (`+$0.02` per completed source, $2.00 ceiling; >2 sources runs async).
- Paginate with `query.page`/`query.perPage` for interactive UIs; unpaginated responses cap at 4,000 records with `capped: true`.

## Errors & limits
- Envelope is `{"error": {"code", "message"}}`; back off on 429 per `Retry-After`. See `errors/offendersearch-problem-types.yml` and `rate-limits/offendersearch-rate-limits.yml`.
