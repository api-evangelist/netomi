---
name: Tune a Netomi bot's rate limits and regression-test its utterances
description: Read and set the per-bot rate-limit windows, then run a bulk test-utterance
  batch against the bot and poll for results.
api: openapi/netomi-agentdesk-openapi.json
operations: [fetchRateConfig, saveRateConfig, updatedRateConfig, deleteRateConfig,
  submitRequest, pollScript, postRequest]
generated: '2026-08-01'
method: generated
source: openapi/netomi-agentdesk-openapi.json
---

# Tune rate limits and regression-test utterances

Two operational chores an agent can safely automate against a Netomi bot: the
rate-limit configuration surface, and the bulk test-utterance harness.

## Part 1 — rate limits

All four verbs live on the same path, `/v1/rate-limit`, and take `botRefId` and/or
`botId` as query parameters.

| Step | operationId | HTTP |
|---|---|---|
| Read current config | `fetchRateConfig` | `GET /v1/rate-limit` |
| Create config | `saveRateConfig` | `POST /v1/rate-limit` |
| Update config | `updatedRateConfig` | `PUT /v1/rate-limit` |
| Remove config | `deleteRateConfig` | `DELETE /v1/rate-limit` |

The body is a `RateLimitConfigRequestDto` with six independent int64 windows:
`minuteLimit`, `hourLimit`, `dayLimit`, `weekLimit`, `monthLimit`, `yearLimit`.

**Always `fetchRateConfig` first.** Netomi publishes no default or maximum values
anywhere, so the only way to know the current limits is to read them. Write back the
full object — a partial write may clear windows you did not intend to change.

## Part 2 — test utterances

1. **Submit the batch.** `submitRequest` — `POST /v2/test-utterances/bulk`.
2. **Poll for results.** `pollScript` — `POST /v2/test-utterances/bulk/poll`.
   Same async submit-then-poll shape as the conversation engine.
3. **Spot-check a single utterance** with `postRequest` — `POST /v1/nlu/predict` — to
   see the intent and entity prediction for one input without running a batch.

## Rules

- `DELETE /v1/rate-limit` removes the bot's limit configuration outright. Confirm the
  `botRefId` before calling it; there is no soft-delete and no undo in the API.
- These operations are **destructive/config-changing** and have no idempotency key.
  Do not retry a failed `saveRateConfig` blindly — re-read with `fetchRateConfig` and
  reconcile.
- Branch on `exceptionCode` in the `ServiceResponse` envelope, not the HTTP status.
  Relevant codes: `DATA_NOT_FOUND`, `INVALID_BOT_DETAILS`, `INVALID_CONFIGURATION`,
  `INVALID_CONF`, `INSUFFICIENT_PERMISSION`, `RATE_LIMIT_EXCEED_ERROR`,
  `THROTTLE_ERROR`, `REQUEST_PAYLOAD_MAX_SIZE_EXCEED_ERROR`.
- The recommended execution posture for these write operations is in
  `agentic-access/netomi-agentic-access.yml`.
