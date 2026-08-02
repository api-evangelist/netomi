---
name: Authorize a Netomi visitor session
description: Exchange an external auth token or auth code for an authorized Netomi
  visitor, then confirm the platform is operational before starting a conversation.
api: openapi/netomi-agentdesk-openapi.json
operations: [messagesPost_4, statusAPI, healthCheck]
generated: '2026-08-01'
method: generated
source: openapi/netomi-agentdesk-openapi.json
---

# Authorize a Netomi visitor session

Use this when the bot is configured for **authenticated** sessions and you need to
establish a known end user rather than an anonymous guest.

## Steps

1. **Authorize the visitor.**
   `messagesPost_4` — `POST /v1/authorize/visitors`.
   Body is an `ExternalAuthenticationRequest`:

   | field | meaning |
   |---|---|
   | `authToken` | Your externally issued token for the user |
   | `authCode` | Alternative to `authToken` — an exchangeable code |
   | `clientId` | Your Netomi client identifier |
   | `visitorKey` | Your stable identifier for the end user |
   | `timestamp` | Request timestamp |
   | `expiryTime` | Integer (int64) expiry |

   Netomi's SDK guidance is explicit: **generate this material server-side**, scope it
   to the user, and keep it short-lived. Never embed a long-lived credential in a
   client app.

2. **Check the platform is operational for your bot.**
   `statusAPI` — `GET /v1/platform/status`.
   This operation requires two headers the rest of the API does not:
   `X-Hub-Signature` and `X-Client-Id`. The response is a `PlatformStatusResponse`
   whose `status` is one of `FULLY_OPERATIONAL`, `MAINTENANCE`, `INCIDENT`, `OUTAGE`,
   `INOPERATIVE`. Abort the flow on `OUTAGE` or `INOPERATIVE`.

3. **Liveness only.** `healthCheck` — `GET /ceaas/v1/health/{secretId}` is a
   service liveness probe, not an auth or tenancy check. Do not use it to decide
   whether a session is valid.

## Rules

- The spec declares **no `securitySchemes`**, so nothing about credentials is
  machine-readable. Treat `authentication/netomi-authentication.yml` as the profile
  of record.
- Error codes specific to this flow: `AUTHENTICATION_FAILED_ERROR`,
  `AUTHORIZATION_FAILED_ERROR`, `BAD_CREDENTIAL`, `ACCESS_TOKEN_EXPIRED`,
  `UN_AUTHORISED_ERROR`, `UN_AUTHORISED_ACCESS`, `UNAUTHORIZED_API_REQUEST`,
  `ILLEGEAL_TOKEN_HEADER` (sic — spelled that way in the enum), `INVALID_ROLE`,
  `INSUFFICIENT_PERMISSION`, `ACCESS_LOCKED`, `ATTEMPTS_EXCEEDED`,
  `USER_ACCOUNT_TEMPORARY_BLOCKED`, `CLIENT_API_CREDENTIAL_MISSING_ERROR`,
  `CLIENT_API_CREDENTIAL_DOES_NOT_EXIST_ERROR`.
- **`ACCESS_LOCKED` / `ATTEMPTS_EXCEEDED` mean stop, not retry.** Repeating the call
  will keep the account locked.
- On the mobile SDKs the equivalent of re-auth is the reauthorization event loop: the
  SDK raises `reauthorizationRequest`, your app answers with
  `sendEventToSdk(type: .reauthorizationSuccess, jwt:)` carrying a **fresh** JWT.
  Answering without a new JWT is the documented cause of a reauthorization loop.
