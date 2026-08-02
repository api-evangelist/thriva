---
name: Handle clinical escalations
description: >-
  Safety-critical flow: receive Thriva's clinical-escalation webhook for an out-of-range
  result and acknowledge it before the deadline.
api: openapi/thriva-platform-api-openapi.yml
operations:
  - POST /oauth/token
  - GET /v1/result-sets/{result_set_id}
  - POST /v1/escalations/{escalation_id}/acknowledge
generated: '2026-07-21'
method: generated
---

# Handle clinical escalations

The spec declares no operationIds; operations are referenced as `METHOD /path` exactly as
published in `openapi/thriva-platform-api-openapi.yml`.

Clinical escalations are raised by Thriva's Escalation Calculator (a regulated medical
device — partner instructions for use are distributed via the Clinical Escalations guide,
https://docs.thriva.io/docs/clinical-escalations).

1. **Listen for the webhook.** `result_set.escalation_raised` (Svix-signed — verify
   `svix-signature` before trusting it, see `asyncapi/thriva-webhooks.yml`). The payload
   carries `escalation_id`, `level` (e.g. `very_high`), `result_set_id`, `test_id`,
   `order_id` and `acknowledgment_due_by` (ISO 8601 deadline).
2. **Review the result.** `GET /v1/result-sets/{result_set_id}` with
   `include=biomarker_results,escalations` to see which biomarker escalated and why.
3. **Acknowledge in time.** `POST /v1/escalations/{escalation_id}/acknowledge` **before**
   `acknowledgment_due_by`. This is the contractual signal that the partner has taken
   clinical responsibility for the escalated result.

## Rules

- Treat this flow as safety-critical: escalations represent potentially dangerous blood
  results. Never auto-dismiss; route to a clinician and only acknowledge once the
  escalation is actually being handled.
- The spec declares 422 errors for archived escalations and invalid escalation levels
  (`ArchivedEscalationError`, `InvalidEscalationLevelError` schemas) — an acknowledge call
  can fail if the escalation is already archived.
- Test the flow in sandbox with the `LAB HH` (too high) and `LAB LL` (too low) postcodes
  (`sandbox/thriva-sandbox.yml`).
