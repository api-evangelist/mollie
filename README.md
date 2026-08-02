# Mollie

Mollie B.V. is a Dutch payment service provider headquartered in Amsterdam that lets
businesses across Europe accept online, point-of-sale and recurring payments through a
single REST API and hosted checkout.

- Website: https://www.mollie.com/
- Developer docs: https://docs.mollie.com/
- API reference: https://docs.mollie.com/reference/overview
- GitHub: https://github.com/mollie
- Status: https://status.mollie.com/

## APIs

| API | Base URL | Contract |
|---|---|---|
| Mollie API | https://api.mollie.com | OpenAPI 3.1.0, 124 operations, 33 API groups |
| Mollie MCP Server | https://mcp.mollie.com/mcp | Hosted MCP over HTTP, OAuth-gated |

## Artifacts

| Directory | What it holds |
|---|---|
| `openapi/` | Mollie's own OpenAPI 3.1 document, harvested from github.com/mollie/openapi |
| `overlays/` | API Evangelist Overlay 1.0.0 of our observations over that spec |
| `authentication/` `scopes/` | Bearer/API-key/OAuth profile and the 61 published OAuth scopes |
| `conventions/` | Idempotency, pagination, versioning, error envelope, rate-limit signalling |
| `errors/` | Error catalogue (HAL, not RFC 9457) and 119 payment decline / status-reason codes |
| `asyncapi/` | Webhook catalogue and the 36 published event types (no AsyncAPI document) |
| `sandbox/` | Test keys, test cards, magic amounts, transfer simulation scenarios |
| `mcp/` | MCP server manifest and the tool ↔ operationId crosswalk |
| `skills/` | Mollie's own published Agent Skill plus four generated flow skills |
| `packages/` | Official SDKs across PHP, Node/TypeScript, Python, Ruby, Go, Java and C# |
| `components/` | Mollie.js, Mollie Components, Checkout Sessions, wallet integrations |
| `data-model/` | Entity graph and identifier prefixes derived from the spec |
| `security/` `well-known/` | security.txt, OAuth discovery, domain security, disclosure and trust posture |
| `conformance/` `lifecycle/` `changelog/` | Standards conformance, versioning/deprecation posture, recent changes |
