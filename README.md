# Netomi

Netomi (founded 2016 as msg.ai) is an enterprise agentic AI platform for customer
experience. Its "Agentic OS for CX" orchestrates a network of AI agents across chat,
email, telephony, social, search, MCP and API channels, with a governance layer
(topic and policy guardrails, prompt security, response validation) over a
task-planning orchestration agent.

- Website: https://www.netomi.com/
- Status: https://status.netomi.com
- GitHub: https://github.com/msgai
- Console: https://studio.netomi.com (Agentic Studio, account required)
- Docs portal: https://docs.netomi.com/ (login required)

## Public API surface

Netomi publishes a **live but unlinked OpenAPI 3.1.0** — the *AgentDesk REST API*,
56 paths / 67 operations / 103 schemas — at `https://api.netomi.com/v3/api-docs`, with
Swagger UI at `https://api.netomi.com/swagger-ui.html`. It is not referenced from any
Netomi documentation and was found only by probing Springdoc default paths after
`/openapi.json`, `/swagger.json`, `/api-docs` and `/docs` all 404'd. It declares **no
securitySchemes, no operation summaries or descriptions, no examples and only 200
responses**, so the contract exists but carries almost no authored content.

No GraphQL, AsyncAPI, gRPC, MCP server or A2A Agent Card was found on any Netomi host
(see `well-known/` and `conformance/`). Alongside the spec, the public surface is the
first-party Mobile Chat SDK and the platform status feed:

| Surface | Where |
|---|---|
| Platform API | `https://api.netomi.com` — OpenAPI at `/v3/api-docs`; conversation engine, NLU, rate-limit config, and 16 inbound channel webhooks |
| Status API | `https://status.netomi.com/api/v2` — Atlassian Statuspage v2 JSON + Atom/RSS |
| iOS SDK | SPM `github.com/msgai/netomi-chat-ios` 1.29.2 (CocoaPods `NetomiChatSDK` deprecated, ends 2026-10-01) |
| Android SDK | Maven Central `com.netomi.chat:chat-widget-android` 1.26.0 |
| React Native | npm `@netomi.com/netomi-chat-react-native` 1.7.0 |

Authentication is a per-tenant `botRefId` plus an optional caller-supplied **JWT** for
authenticated sessions, with a documented reauthorization flow. Production runs in
US, EU and SG regions.

> `docs.netomi.site` is an unrelated project of the same name (a zero-knowledge payment
> network on an unmodified GitBook template) and is **not** a Netomi Inc. property.
