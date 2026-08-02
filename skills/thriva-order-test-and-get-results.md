---
name: Order an at-home test and retrieve results
description: >-
  End-to-end happy path on the Thriva Platform API: authenticate, create a User, place an
  Order for a test kit, track fulfilment, and retrieve the biomarker results when the lab
  has processed the sample.
api: openapi/thriva-platform-api-openapi.yml
operations:
  - POST /oauth/token
  - POST /v1/users
  - GET /v1/test-profiles
  - POST /v1/orders
  - GET /v1/orders/{order_id}
  - GET /v1/orders/{order_id}/tests
  - GET /v1/result-sets/{result_set_id}
generated: '2026-07-21'
method: generated
---

# Order an at-home test and retrieve results

The spec declares no operationIds; operations are referenced as `METHOD /path` exactly as
published in `openapi/thriva-platform-api-openapi.yml`.

1. **Authenticate.** `POST /oauth/token` with your Thriva-issued `clientId` and `secret`
   (OAuth 2.0 client-credentials). Include the `audience` parameter — for sandbox, the
   audience is `https://api.euw2.sandbox.thriva.io/` and the auth host is
   `https://auth.sandbox.thriva.io/`. Use the returned access token as a Bearer token on
   every call. See `authentication/thriva-authentication.yml`.
2. **Pick a test profile.** `GET /v1/test-profiles` (optionally `include=biomarkers`) to see
   the catalogue of profiles Thriva can analyse.
3. **Create the test subject.** `POST /v1/users` with first/last name, sex and date of birth
   (labs verify identity and reference ranges depend on sex and age). Store the Thriva user
   ID — an optional `external_reference` can carry your own ID, but requests must use
   Thriva's.
4. **Place the order.** `POST /v1/orders` with the user, test profile(s) and delivery
   address. In sandbox, the delivery-address postcode selects the scenario (`LAB S` success,
   `LAB F`/`LAB PF` biomarker failures, `LAB HH`/`LAB LL` critical escalations) — see
   `sandbox/thriva-sandbox.yml`.
5. **Track progress.** Prefer webhooks (`fulfillment_order.fulfilled`,
   `test.received_at_lab`, `tracking_event.created` — Svix-signed, see
   `asyncapi/thriva-webhooks.yml`); or poll `GET /v1/orders/{order_id}` and
   `GET /v1/orders/{order_id}/tests`.
6. **Fetch results.** On `result_set.available` (or `result_set.partial_available`), call
   `GET /v1/result-sets/{result_set_id}` with
   `include=test_profiles,biomarker_results,escalations`.

## Rules

- The API is JSON:API-shaped: paginate lists with `page[number]`/`page[size]` (10–100) and
  expand relationships with `include=` (see `conventions/thriva-conventions.yml`).
- Errors are JSON:API error objects (`title`, `detail`, `source.pointer`); 415 means a wrong
  content type (see `errors/thriva-problem-types.yml`).
- No idempotency-key contract exists — do not blindly retry `POST /v1/orders`; reconcile
  with `GET /v1/orders` first.
- If a result raises a clinical escalation, follow the escalation skill —
  acknowledgment has a hard deadline.
