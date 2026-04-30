---
title: Idempotency Rules
owner: API Architecture
status: production-ready
last_reviewed: 2026-04-30
tags: [api, idempotency, payments, offline-sync, caching]
source: Unified Commerce production scope + database design
---

# Idempotency Rules

## Purpose

This file defines idempotency rules for retryable API operations.

Idempotency prevents duplicate payments, duplicate sales, duplicate receipts, duplicate orders, duplicate refunds and duplicate offline sync records.

Read with:

- [[04-api/concurrency-rules]]
- [[04-api/offline-sync-api-rules]]
- [[04-api/payment-refund-api-rules]]
- [[05-backend/caching-strategy]]
- [[03-data/indexing-strategy]]

---

## Core rules

- Require idempotency for payments, refunds, orders, offline sync, receipt generation and retryable sales.
- Store idempotency keys tenant-scoped.
- Return original result for safe duplicate retries.
- Do not depend on application cache for idempotency correctness.
- Use PostgreSQL uniqueness or lookup as the final guard.

---

## Idempotency must not depend on cache

Do not check idempotency only in backend cache.

Reason:

- Cache can expire.
- Cache can be evicted.
- Cache can be missing after deployment restart.
- Cache can be wrong if tenant key is missing.
- Payment and offline sync duplicates are too risky.

Correct approach:

```text
API request -> PostgreSQL idempotency/unique check -> source transaction -> response
```

Incorrect approach:

```text
API request -> memory cache says not seen -> create payment/sale -> duplicate risk
```

---

## Idempotency areas

| Area | Idempotency input | Database guard |
|---|---|---|
| Payment | `idempotency_key` or `client_payment_id` | `payments(tenant_id, idempotency_key)` or client payment unique rule. |
| POS offline sale | `client_transaction_id`, `client_sale_id` | `sales` and offline sync item/queue unique rules. |
| Offline sync item | `client_entity_id` | `offline_sync_items(tenant_id, device_id, entity_type, client_entity_id)`. |
| Receipt generation | document reference + receipt number/client receipt id | `receipts` uniqueness. |
| Refund | original payment + idempotency context | refund/payment DB validation. |
| Order placement | client/order idempotency context where supported | order/source cart validation. |

---

## Payment idempotency

Payment idempotency must protect:

- Card/manual reference retry.
- QR/manual reference retry.
- Gateway timeout retry.
- Duplicate cashier click.
- Offline payment sync retry.
- Refund retry.

The `payments.idempotency_key` field exists for tenant-scoped idempotency guard.

Do not store payment idempotency only in cache.

---

## Offline sync idempotency

Offline sync must protect:

- Replayed sync batch.
- Replayed sync item.
- Replayed sale payload.
- Replayed payment payload.
- Browser/device resend after reconnect.

Relevant tables:

- `offline_sync_batches`
- `offline_sync_items`
- `offline_sale_sync_queue`
- `offline_payment_sync_queue`
- `sales`
- `payments`

Offline sync idempotency must be tenant/device/client-id aware.

---

## Document sequence rule

Document sequence allocation is not idempotency cache.

`document_sequences` must be handled with database row-level locking or equivalent safe transaction handling.

Never cache the next number for:

- Sales.
- Orders.
- Returns.
- Exchanges.
- Receipts.
- Deliveries.
- Stock transfers.
- Stock adjustments.

---

## API behavior for duplicate retry

| Case | Required behavior |
|---|---|
| Same idempotency key, same payload | Return original result where safe. |
| Same idempotency key, different payload | Reject as idempotency conflict. |
| Same offline client entity ID | Return accepted/rejected/conflict state. |
| Same payment already captured | Return existing payment state where safe. |
| Same receipt already generated | Return existing receipt reference. |

---

## Cache boundary

Cache may store recent result for faster response, but only after DB idempotency has been recorded.

Allowed:

```text
DB-backed idempotency result -> optional short-lived response cache
```

Not allowed:

```text
cache-only idempotency tracking
```

---

## Checklist

Before implementing a retryable API:

- [ ] Define idempotency key source.
- [ ] Include `tenant_id` in uniqueness logic.
- [ ] Include device/client IDs where offline POS is involved.
- [ ] Store/check idempotency in PostgreSQL.
- [ ] Handle same-key different-payload conflict.
- [ ] Return original result for safe retry.
- [ ] Add tests for duplicate request.
- [ ] Add tests for cache miss after first successful request.

---

## Related files

- [[04-api/concurrency-rules]]
- [[04-api/offline-sync-api-rules]]
- [[04-api/payment-refund-api-rules]]
- [[05-backend/caching-strategy]]
