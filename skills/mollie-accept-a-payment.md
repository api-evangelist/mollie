---
name: mollie-accept-a-payment
description: Create a Mollie payment, redirect the shopper to the hosted checkout, and confirm the result before fulfilling the order.
api: openapi/mollie-openapi-original.yml
operations: [create-payment, get-payment, list-methods, create-webhook, get-webhook-event]
generated: '2026-08-01'
method: generated
source: openapi/mollie-openapi-original.yml + https://docs.mollie.com/docs/accepting-payments
---

# Accept a payment with Mollie

Use this when an agent has to take a one-off payment from a shopper.

## Auth

Send `Authorization: Bearer <key>` against `https://api.mollie.com`.

- A profile API key (`test_…` / `live_…`) — the key itself selects the mode.
- Or an advanced access token (`access_…`) / OAuth token, plus `testmode=true` for test mode.
  OAuth needs the `payments.write` scope to create and `payments.read` to read.

Always develop against a `test_` key first. See `sandbox/mollie-sandbox.yml` for test cards
and the magic EUR amounts that force specific failure reasons.

## Steps

1. **(Optional) show the methods you can offer.** `list-methods` — `GET /v2/methods` with
   `amount[currency]`, `amount[value]` and `locale` returns only the methods enabled on the
   profile and valid for that amount. Do not hardcode a method list; availability varies by
   country, currency and amount.

2. **Create the payment.** `create-payment` — `POST /v2/payments` with at minimum
   `amount` (`{currency, value}` where `value` is a decimal *string*, e.g. `"10.00"`),
   `description`, and `redirectUrl`. Add `webhookUrl` if you are using classic webhooks,
   `method` to skip the method picker, and `metadata` to carry your own order reference.
   Send an `Idempotency-Key` header (UUID v4) so a network retry cannot create two payments.

3. **Redirect the shopper.** Send them to `_links.checkout.href` from the response. In test
   mode this is the test checkout screen, where you can force any final status.

4. **Wait for the event, do not poll.** Either set `webhookUrl` on the payment, or
   subscribe once with `create-webhook` — `POST /v2/webhooks` with a `url` and
   `eventTypes` such as `payment.paid`, `payment.failed`, `payment.expired`,
   `payment.canceled`. Reply `200` fast and process asynchronously.

5. **Re-read before you fulfil.** Never trust the webhook body as the source of truth.
   Call `get-payment` — `GET /v2/payments/{paymentId}` and fulfil only when
   `status == "paid"`. `authorized` means funds are held, not captured. Use
   `get-webhook-event` — `GET /v2/events/{webhookEventId}` if you need to inspect what was
   delivered.

## Rules

- `amount.value` is a string with two decimals. `10` and `10.0` are invalid.
- The shopper returning to `redirectUrl` does **not** mean the payment succeeded — they may
  have closed the issuer page. Status comes from `get-payment` only.
- On `429`, back off for `Retry-After` seconds; read `RateLimit` / `RateLimit-Policy` on
  every response to pace yourself (`conventions/mollie-conventions.yml`).
- On failure, read `statusReason.code` against `errors/mollie-decline-codes.yml`. Do not
  show issuer-specific reasons such as `suspected_fraud`, `lost_card` or `stolen_card` to
  the shopper.
- Errors are `application/hal+json` with `status`, `title`, `detail` and often `field` —
  not RFC 9457. See `errors/mollie-problem-types.yml`.
- `503` on `create-payment` usually means the method supplier (e.g. the iDEAL network) is
  down. Retry later with the same `Idempotency-Key`.
