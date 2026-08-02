---
name: Bulk-order kits and register samples
description: >-
  Clinic/site flow on the Thriva Platform API: ship a batch of unassigned kits to one
  address, validate sample references, then bind each collected sample to a User as a Test.
api: openapi/thriva-platform-api-openapi.yml
operations:
  - POST /oauth/token
  - POST /v1/bulk-orders
  - GET /v1/bulk-orders
  - GET /v1/bulk-orders/{bulk_order_id}
  - GET /v1/bulk-orders/sample-references/{sample_reference}
  - POST /v1/bulk-orders/tests
  - GET /v1/tests/{test_id}
generated: '2026-07-21'
method: generated
---

# Bulk-order kits and register samples

The spec declares no operationIds; operations are referenced as `METHOD /path` exactly as
published in `openapi/thriva-platform-api-openapi.yml`.

1. **Authenticate** with `POST /oauth/token` (client credentials + `audience`), Bearer token
   on all calls.
2. **Ship kits in bulk.** `POST /v1/bulk-orders` creates and sends a Bulk Order of kits to a
   specified address (a clinic, workplace or event site). Monitor with
   `GET /v1/bulk-orders` and `GET /v1/bulk-orders/{bulk_order_id}`.
3. **Validate the sample reference before use.**
   `GET /v1/bulk-orders/sample-references/{sample_reference}` returns the details and status
   of a sample reference from a Bulk Order — use it to check a reference is valid before
   creating a Test from it.
4. **Bind sample to person.** `POST /v1/bulk-orders/tests` creates a Test from a sample
   reference and links it with a User (create the User first via `POST /v1/users` if
   needed).
5. **Follow the sample.** Track lab receipt and results via webhooks
   (`test.received_at_lab`, `result_set.available`) or `GET /v1/tests/{test_id}`.

## Rules

- Validate every scanned `sample_reference` (step 3) before `POST /v1/bulk-orders/tests` —
  it is the documented pre-check for invalid references.
- Lists are JSON:API paginated (`page[number]`/`page[size]`, 10–100 per page).
- No idempotency-key contract: on a failed/timeout `POST /v1/bulk-orders/tests`, check the
  sample reference status before retrying.
