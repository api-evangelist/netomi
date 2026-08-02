---
name: Ask Netomi and collect the response
description: Submit a customer utterance to the Netomi conversation engine and collect
  the asynchronous agent response, then read prior turns for context.
api: openapi/netomi-agentdesk-openapi.json
operations: [conversationEngineAsync, getResponse, getConversationHistory]
generated: '2026-08-01'
method: generated
source: openapi/netomi-agentdesk-openapi.json
---

# Ask Netomi and collect the response

The Netomi conversation engine is **asynchronous**: you post a message and then poll
for the answer using the same `requestId`. Do not expect the reply on the POST.

## Before you start

- You need a **`botRefId`** (your Netomi bot reference id) and, for authenticated
  bots, a caller-supplied **JWT**. See `authentication/netomi-authentication.yml`.
- Base URL is `https://api.netomi.com`. The harvested spec advertises `http://` — that
  is a Springdoc "generated server url" artefact; always call HTTPS.
- The spec declares **no `securitySchemes`**, so the credential material is not
  described machine-readably. Send what your Netomi onboarding gave you.

## Steps

1. **Submit the utterance.**
   `conversationEngineAsync` — `POST /v2/messages`.
   The request body is `application/json`. Keep your own `requestId`; you need it to
   collect the answer.

2. **Collect the response.**
   `getResponse` — `GET /v2/messages?requestId={requestId}`.
   `requestId` is a **required** query parameter. Poll until the engine has produced a
   reply. The payload is a `ConversationResponse`, which carries `Message` objects
   (38 possible payload shapes — text, cards, carousels, receipts, forms, airline
   templates), plus `Session`, `Visitor`, `Intent`, `AgentInfo` and `BotMetaInfo`.
   See `data-model/netomi-data-model.yml` for the entity graph.

3. **Read prior turns when you need context.**
   `getConversationHistory` — `GET /v2/webhook/history/{conversationId}`.
   Optional bounds: `numberOfDays`, `numberOfMessages`, `timestamp`.

4. **Prefer the structured response when you are parsing programmatically.**
   `getStructuredResponse` — `GET /v3/ceaas/messages`.

## Rules

- **No idempotency contract.** Netomi documents no idempotency key and no
  de-duplication window (`conventions/netomi-conventions.yml`). Re-POSTing the same
  utterance may create a second turn — reuse your `requestId` and poll instead of
  retrying the POST.
- **Every response is the same envelope.** Success and failure both come back as
  `ServiceResponse` — `{statusCode, statusMessage, exceptionCode}` — often with
  HTTP 200. **Branch on `exceptionCode`, not on the HTTP status.**
- Error codes to expect on this flow: `API_EMPTY_PAYLOAD_ERROR`, `INVALID_INPUT_ERROR`,
  `INVALID_BOT_DETAILS`, `CONVERSATION_NULL`, `STANDARD_RESPONSE_EMPTY`,
  `CONNECTION_TIME_OUT_ERROR`, `THROTTLE_ERROR`, `RATE_LIMIT_EXCEED_ERROR`.
  Full registry: `errors/netomi-error-codes.yml`.
- On `THROTTLE_ERROR` or `RATE_LIMIT_EXCEED_ERROR`, back off. Limits are configured
  per bot across minute/hour/day/week/month/year windows and Netomi publishes no
  default values — read them with `fetchRateConfig` (`GET /v1/rate-limit`) rather than
  assuming.
- The spec declares only `200` responses. Treat undeclared 4xx/5xx as possible.
