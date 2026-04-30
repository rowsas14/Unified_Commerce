---
title: Entity Relationship Map
folder: 03-data
status: production-ready
owner: Architecture / Backend / Data
tags: [relationships, erd]
---

# Entity Relationship Map

This file shows the main cross-module entity relationships. Detailed PK/FK information lives under `entities/*`.

---

## Main operational ER flow

```mermaid
erDiagram
    tenants ||--o{ outlets : owns
    tenants ||--o{ users : owns
    tenants ||--o{ roles : owns
    roles ||--o{ role_permissions : grants
    tenants ||--o{ products : owns
    products ||--o{ product_variants : has
    outlets ||--o{ inventory_balances : holds
    product_variants ||--o{ inventory_balances : tracked_as
    product_variants ||--o{ stock_movements : moves
    outlets ||--o{ pos_devices : assigns
    tills ||--o{ till_sessions : opens
    till_sessions ||--o{ sales : contains
    sales ||--o{ sale_lines : has
    sales ||--o{ sale_payment_allocations : paid_by
    payments ||--o{ sale_payment_allocations : allocated
    customers ||--o{ carts : owns
    carts ||--o{ cart_items : contains
    carts ||--o{ orders : converts_to
    orders ||--o{ order_items : has
    orders ||--o{ deliveries : fulfilled_by
    sales ||--o{ returns : returned_from
    orders ||--o{ returns : returned_from
    returns ||--o{ exchanges : may_create
    receipts ||--o{ receipt_print_logs : logs
    pos_devices ||--o{ offline_sync_batches : syncs
```

---

## Relationship hotspots

| Hotspot | Why it matters |
|---|---|
| Product → variant → stock/price/sale/order | Variant is the sellable unit. |
| Sale → payment → receipt → stock movement | POS completion must be transactional. |
| Order → reservation → fulfillment → payment | E-Commerce workflow needs separate stock/payment/fulfillment states. |
| Return/exchange → refund/payment → stock movement | Post-sale workflows affect money and inventory. |
| Device → sync batch → sync item → accepted source tables | Offline POS requires dedupe and conflict handling. |
| Tenant feature entitlement → role feature assignment → permission | Feature access is not a simple role check. |

---

## Rules for AI IDE tools

- Do not create new relationships without checking the owning entity file.
- Do not infer a direct relationship when a mapping/allocation table exists.
- Do not combine sale and order payment allocations.
- Do not treat offline queues as completed transactions.
- Do not read reporting summaries as source-of-truth financial records.

Related: [[data-dictionary-index]], [[schema-principles]], [[offline-sync-data-model]].
