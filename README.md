# Mollie

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
