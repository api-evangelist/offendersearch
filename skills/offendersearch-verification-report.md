---
generated: '2026-09-03'
method: generated
name: Produce a verification report PDF
description: Turn a search you already ran into a branded, timestamped PDF with a source citation on every record.
api: openapi/offendersearch-openapi.json
operations: [syncSearch, makeReport, getProofDoc]
source: >-
  Grounded in openapi/offendersearch-openapi.json (operationIds verified) and
  https://offendersearch.app/docs/reports.md.
---

# Produce a verification report

A stored, exportable proof document for compliance and audit trails.

## Auth
- `X-API-Key` header; any valid key can call the report endpoint (no per-key entitlement). Billing must be enabled (402 otherwise). Each document bills $0.02.

## Steps
1. **Run the search first** — `syncSearch` (`POST /v1/search`); keep the returned `searchId`.
2. **Request the report** — `makeReport` (`POST /v1/report`) with the `searchId` of a search run within the last 7 days. The response points at the rendered document.
3. **Fetch the document** — `getProofDoc` (`GET /v1/proof-docs/{token}`) retrieves the rendered PDF: every match and field, timestamped, with a source citation on every record.

## Rules
- The 7-day window is on the source search, so generate reports promptly after screening.
- `makeProof` (`POST /v1/searches/{searchId}/proof`) is an internal per-registry look-alike add-on, NOT the verification report — use `makeReport`.
