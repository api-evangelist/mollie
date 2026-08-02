---
name: mollie-refund-and-reconcile
description: Refund a Mollie payment safely, handle chargebacks, and reconcile payouts against settlements and balance transactions.
api: openapi/mollie-openapi-original.yml
operations: [get-payment, create-refund, get-refund, cancel-refund, list-all-refunds, list-all-chargebacks, list-settlements, list-settlement-payments, list-settlement-refunds, list-settlement-chargebacks, list-balances, list-balance-transactions, get-balance-report, list-payouts]
generated: '2026-08-01'
method: generated
source: openapi/mollie-openapi-original.yml + https://docs.mollie.com/docs/refunds
---

# Refund and reconcile with Mollie

## Auth

`Authorization: Bearer <key>`. OAuth scopes: `payments.read`, `refunds.read`,
`refunds.write`, `settlements.read`, `balances.read`, `balance-reports.read`,
`payouts.read`.

## Refunding

1. **Check the payment first.** `get-payment` — `GET /v2/payments/{paymentId}`. Read
   `amountRemaining` to see what is still refundable. Refunding more than remains is
   rejected.

2. **Create the refund.** `create-refund` — `POST /v2/payments/{paymentId}/refunds` with
   `amount` and `description`. **Send an `Idempotency-Key`** — two identical refund
   requests in short succession on the same payment return `409 Conflict`, and without a
   key a retry can create two separate partial refunds.

3. **Track it.** `get-refund` — `GET /v2/payments/{paymentId}/refunds/{refundId}`. Statuses
   run `queued` → `pending` → `processing` → `refunded`, or `failed` / `canceled`. Refunds
   execute after a two-hour delay.

4. **Cancel if you were wrong.** `cancel-refund` —
   `DELETE /v2/payments/{paymentId}/refunds/{refundId}` works only while the status is
   `queued` or `pending`.

5. **Report across the account.** `list-all-refunds` — `GET /v2/refunds` returns refunds
   organization-wide instead of per payment.

## Chargebacks

`list-all-chargebacks` — `GET /v2/chargebacks` lists disputes across the account.
Subscribe to `chargeback.received` and `chargeback.reversed`
(`asyncapi/mollie-webhooks.yml`); a chargeback is a debit against the balance, so it must
land in your ledger even though you never initiated it.

## Reconciling

1. `list-settlements` — `GET /v2/settlements` for the settlement periods. Use
   `get-open-settlement` / `get-next-settlement` for the in-flight ones.
2. For one settlement, pull all four sides:
   `list-settlement-payments`, `list-settlement-captures`, `list-settlement-refunds`,
   `list-settlement-chargebacks` under `/v2/settlements/{settlementId}/…`. A settlement
   nets payments minus refunds, chargebacks and fees.
3. `list-balances` / `list-balance-transactions` —
   `GET /v2/balances/{balanceId}/transactions` is the transaction-level ledger.
   `get-balance-report` — `GET /v2/balances/{balanceId}/report` gives grouped totals for a
   date range; `422` means your `from` is after your `until`.
4. `list-payouts` — `GET /v2/payouts` for the money actually sent to the bank account.
   Subscribe to `payout.completed` and `payout.failed`.

## Rules

- Every list endpoint is cursor-paginated: `limit` (max 250, default 50), `from`, `sort`,
  and you follow `_links.next`. Do not build page numbers.
- `count` may legitimately be `0` on an empty list.
- Amounts are strings, so total in minor units or a decimal type — never in a float.
- Rate limits are per-policy and dynamic. On `429`, honour `Retry-After`; reconciliation
  jobs are exactly the workload that trips them.
