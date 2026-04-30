---
title: Concurrency Rules
owner: API Architecture
status: production-ready
last_reviewed: 2026-04-30
tags: [api, concurrency, caching]
source: Unified Commerce production scope + database design
---

# Concurrency Rules

## Purpose

This file defines API-level concurrency rules for the Unified Commerce system.

Concurrency rules protect against duplicate payments, overselling, invalid coupon usage, wrong document numbers, conflicting offline sync and stale cached decisions.

Read with:

- [[04-api/idempotency-rules]]
- [[05-backend/transaction-boundary-rules]]
- [[05-backend/caching-strategy]]
- [[03-data/indexing-strategy]]

---

## Core rules

- Use locks for document sequences.
- Protect coupon usage counters.
- Protect stock reservations and balances.
- Prevent duplicate payment capture/refund.
- Do not depend on stale cache during final writes.
- Use PostgreSQL as the final concurrency authority.

---

## Stale cache and concurrent updates

Cache can help API read performance, but it must not decide final transaction outcomes.

| Operation | Cache allowed for display? | Final decision must use DB? |
|---|---:|---:|
| Product search | Yes | Yes before sale/order commit |
| Price display | Yes | Yes before sale/order commit |
| Tax display | Yes | Yes before sale/order commit |
| Stock availability display | Short-lived only | Yes |
| Permission hint | Yes | Yes for sensitive action |
| Payment method display | Yes | Yes for payment creation |
| Coupon availability display | Yes | Yes during redemption |
| Offline sync preview | No | Yes |

---

## Stock concurrency

Stock-related APIs must protect:

- POS sale stock deduction.
- E-commerce stock reservation.
- Reservation release or expiry.
- Return stock movement.
- Exchange stock movement.
- Stock adjustment.
- Stocktake posting.
- Offline sync stock conflicts.

Never accept final stock decisions from cached `inventory_balances`.

---

## Payment concurrency

Payment APIs must protect:

- Duplicate capture requests.
- Duplicate refund requests.
- Retry after timeout.
- Payment allocation to sale/order.
- Refund total exceeding captured amount.
- Exchange difference payment/refund.

Payment idempotency must be backed by PostgreSQL, not by cache alone.

---

## Coupon and discount concurrency

Coupon redemption must protect:

- `max_uses`.
- `max_uses_per_customer`.
- `used_count`.
- Date validity.
- Channel validity.
- Stacking rules.

Discount approval must recheck current request state.

A cached discount request state must not approve/reject a stale workflow.

---

## Document sequence concurrency

`document_sequences` must be handled with database locking.

Required for:

- Sale number.
- Order number.
- Return number.
- Exchange number.
- Receipt number.
- Transfer number.
- Stock adjustment number.
- Delivery number.

Never cache the next sequence number.

---

## Offline sync concurrency

Offline sync APIs must use DB-backed duplicate checks for:

- `client_transaction_id`.
- `client_sale_id`.
- `client_payment_id`.
- `client_entity_id`.
- Device ID.
- Tenant ID.

Offline conflict detection must not rely on cache.

---

## API response behavior

When a concurrency issue occurs, return a controlled error category:

| Situation | Response behavior |
|---|---|
| Duplicate idempotency key | Return original result or safe duplicate response. |
| Stock conflict | Return stock conflict response or create sync conflict. |
| Coupon limit exceeded | Reject with validation/business error. |
| Payment already captured | Return original payment state where safe. |
| Sequence lock conflict | Retry internally or return controlled failure. |
| Stale status transition | Reject invalid transition. |

---

## Checklist

Before implementing an API write endpoint:

- [ ] Identify source-of-truth table.
- [ ] Identify unique/idempotency constraints.
- [ ] Identify required transaction boundary.
- [ ] Identify possible stale cache risk.
- [ ] Revalidate permissions for sensitive action.
- [ ] Revalidate tenant/outlet/channel context.
- [ ] Add concurrency tests.
- [ ] Add duplicate retry tests.

---

## Related files

- [[04-api/idempotency-rules]]
- [[05-backend/caching-strategy]]
- [[05-backend/transaction-boundary-rules]]
- [[03-data/indexing-strategy]]
