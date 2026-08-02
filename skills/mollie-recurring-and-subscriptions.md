---
name: mollie-recurring-and-subscriptions
description: Set up a Mollie customer, capture a first payment that establishes a mandate, then charge recurring payments or run a subscription.
api: openapi/mollie-openapi-original.yml
operations: [create-customer, create-customer-payment, list-mandates, get-mandate, create-mandate, create-subscription, list-subscription-payments, cancel-subscription, revoke-mandate]
generated: '2026-08-01'
method: generated
source: openapi/mollie-openapi-original.yml + https://docs.mollie.com/docs/recurring-payments
---

# Recurring payments and subscriptions with Mollie

Recurring money movement is the highest-risk flow in this API. Every write step below
**must** carry an `Idempotency-Key`.

## Auth

`Authorization: Bearer <key>`. OAuth scopes: `customers.write`, `mandates.read`,
`mandates.write`, `subscriptions.write`, `payments.write`.

## Steps

1. **Create the customer.** `create-customer` — `POST /v2/customers` with `name`, `email`,
   optional `locale` and `metadata`. Keep the returned `cst_…` id on your side; do not
   re-create a customer you already have.

2. **Take a first payment that establishes the mandate.** `create-customer-payment` —
   `POST /v2/customers/{customerId}/payments` with `sequenceType: "first"`, an `amount`, a
   `description` and a `redirectUrl`. The shopper completes it interactively; that is what
   authorises future charges. (For SEPA Direct Debit you may instead register an existing
   signed mandate with `create-mandate` — `POST /v2/customers/{customerId}/mandates`.)

3. **Confirm the mandate is valid.** `list-mandates` — `GET /v2/customers/{customerId}/mandates`
   or `get-mandate` — `GET /v2/customers/{customerId}/mandates/{mandateId}`. Proceed only
   when a mandate has `status == "valid"`. A `pending` SEPA mandate is not chargeable yet.

4. **Then either charge on demand or subscribe.**
   - **On demand:** `create-customer-payment` with `sequenceType: "recurring"`. This
     executes immediately with no shopper interaction — a blind retry double-charges.
   - **Subscription:** `create-subscription` — `POST /v2/customers/{customerId}/subscriptions`
     with `amount`, `interval` (e.g. `"1 month"`), `description`, optional `times`,
     `startDate`, `webhookUrl` and `mandateId`. Creating it twice bills the customer twice
     for the whole life of both subscriptions.

5. **Reconcile.** `list-subscription-payments` —
   `GET /v2/customers/{customerId}/subscriptions/{subscriptionId}/payments` returns every
   charge Mollie made. Subscribe to `payment.paid` and `payment.failed` events
   (`asyncapi/mollie-webhooks.yml`) rather than polling.

6. **Stop cleanly.** `cancel-subscription` —
   `DELETE /v2/customers/{customerId}/subscriptions/{subscriptionId}` stops future charges.
   `revoke-mandate` — `DELETE /v2/customers/{customerId}/mandates/{mandateId}` withdraws the
   authorisation entirely and stops every subscription that depends on it.

## Rules

- **Idempotency is mandatory here.** Mollie explicitly names recurring payments,
  subscription creation and partial refunds as the flows where a retry causes real harm.
  Reuse the *same* `Idempotency-Key` on a retry; Mollie replays the cached response for one
  hour (`conventions/mollie-conventions.yml`).
- `sequenceType: "recurring"` requires a valid mandate. Without one you get a `422` naming
  the offending field.
- Test-mode recurring payments have no `checkout` URL — they return `changePaymentState`
  so you can force the final status (`sandbox/mollie-sandbox.yml`).
- A failed recurring charge does not cancel the subscription. Handle dunning yourself off
  the `payment.failed` event.
