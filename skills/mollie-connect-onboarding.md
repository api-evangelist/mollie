---
name: mollie-connect-onboarding
description: Onboard a merchant onto a Mollie Connect platform with OAuth, track their onboarding and capabilities, and route or transfer funds on their behalf.
api: openapi/mollie-openapi-original.yml
operations: [create-client-link, oauth-generate-tokens, oauth-revoke-tokens, list-clients, get-client, get-onboarding-status, list-capabilities, get-current-organization, get-partner-status, payment-create-route, create-connect-balance-transfer, list-connect-balance-transfers]
generated: '2026-08-01'
method: generated
source: openapi/mollie-openapi-original.yml + https://docs.mollie.com/docs/getting-started-with-mollie-connect
---

# Mollie Connect: onboard and operate on behalf of merchants

Use this when the caller is a **platform or marketplace** processing payments for other
businesses — not a merchant taking its own payments. Mollie's own integration skill stops
and redirects to Connect in exactly this case.

## Auth

Connect is OAuth 2.0 authorization code with PKCE (`S256`).

- Authorize: `https://my.mollie.com/oauth2/authorize`
- Token / revoke: `https://api.mollie.com/oauth2/tokens`
  (`oauth-generate-tokens` — `POST /oauth2/tokens`, `oauth-revoke-tokens` — `DELETE /oauth2/tokens`)
- Dynamic client registration: `https://api.mollie.com/oauth2/register`
- Metadata: `https://my.mollie.com/.well-known/oauth-authorization-server`

Request only the scopes you use. The full published set is in `scopes/mollie-scopes.yml`;
this flow typically needs `onboarding.read`, `clients.read`, `organizations.read`,
`profiles.write`, `payments.write` and `balance-transfers.write`.

## Steps

1. **Create the onboarding link.** `create-client-link` — `POST /v2/client-links` with the
   merchant's `owner` (name, email) and `address`. A `422` here means a mandatory owner
   field is missing. Send the merchant to the returned link, then run the OAuth
   authorization flow to obtain their access token.

2. **Exchange and store tokens.** `oauth-generate-tokens` with `grant_type=authorization_code`.
   Persist the refresh token; access tokens expire and are refreshed with
   `grant_type=refresh_token`.

3. **Track onboarding to completion.** `get-onboarding-status` — `GET /v2/onboarding/me`
   using the merchant's token. Do not attempt to push data with `submit-onboarding-data`:
   Mollie no longer recommends it and points at Client Links instead.

4. **Check what they can actually do.** `list-capabilities` — `GET /v2/capabilities`
   returns each capability with its requirements and due dates. A capability with an
   overdue requirement is disabled until the requirement is met. Gate your product on this,
   not on a guess.

5. **Enumerate your book.** `list-clients` — `GET /v2/clients` and `get-client` —
   `GET /v2/clients/{organizationId}` from the partner token. `get-partner-status` —
   `GET /v2/organizations/me/partner` confirms your own partnership details.

6. **Split the money.** `payment-create-route` —
   `POST /v2/payments/{paymentId}/routes` splits a payment towards a connected
   organization's balance (delayed routing). For platform-to-merchant movement outside a
   payment, use `create-connect-balance-transfer` —
   `POST /v2/connect/balance-transfers`, and audit with
   `list-connect-balance-transfers`.

## Rules

- Send an `Idempotency-Key` on every write, especially routes and balance transfers.
- Never reuse one merchant's token against another merchant's resources; scope every call
  to the token you were granted.
- `410 Gone` on a profile means it was deleted — remove it from your side rather than
  retrying.
- Beta surfaces in this flow (Capabilities API, Transfers API) may still change shape.
- Business-operations endpoints (Business Accounts, Transfers, Verify Payee) do **not**
  support test mode.
