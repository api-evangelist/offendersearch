---
generated: '2026-09-03'
method: generated
name: Bulk-screen a roster (async, batch, webhooks)
description: Submit high-volume screening with safe retries, then collect by webhook or polling; use batch for row-in/row-out lookups.
api: openapi/offendersearch-openapi.json
operations: [asyncSearch, getSearch, batchSearch]
source: >-
  Grounded in openapi/offendersearch-openapi.json (operationIds verified) and
  https://offendersearch.app/docs/async-and-webhooks.md + /docs/batch.md.
---

# Bulk-screen a roster

Use the asynchronous surface for backfills, periodic re-screens and national sweeps — no request-timeout ceiling, every named jurisdiction runs to completion.

## Auth
- `X-API-Key` header on every request.

## Steps
1. **Submit** — `asyncSearch` (`POST /v1/searches`) with the same body as `/v1/search` plus optional `webhookUrl`. ALWAYS send an `Idempotency-Key` header: a repeated key returns the original job instead of starting (and billing) a second search. Returns 202 with `searchId` and `resultsUrl`.
2. **Collect** — either poll `getSearch` (`GET /v1/searches/{searchId}`) every few seconds until `status` is `complete` or `error`, or receive the finished `SearchResponse` POSTed to your `webhookUrl`.
3. **Verify webhooks** — check `X-Offendersearch-Signature` (HMAC-SHA256 of the raw body with your signing secret, timing-safe compare) and de-duplicate on `searchId` (delivery is at-least-once). See `asyncapi/offendersearch-webhooks.yml`.
4. **Row-in/row-out lookups** — `batchSearch` (`POST /v1/batch`) accepts up to 1,000 lookups per call (JSON or CSV) with per-row fault isolation; each row bills as one call.

## Rules
- Prefer async/batch over thousands of parallel synchronous calls; back off on 429 per `Retry-After`.
- Volume pricing is graduated automatically: $0.15/call, $0.11 past 2,000/month. See `plans/offendersearch-plans-pricing.yml`.
