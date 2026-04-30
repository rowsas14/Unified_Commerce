---
title: {title}
owner: API Architecture / Backend Architecture
status: production-ready
last_reviewed: {DATE}
tags: [{tags}]
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_backend_architecture: Back end Architecture final.txt
source_frontend_architecture: Frontend archi V1.txt
---
# Order Workflow API Rules

Order workflow APIs control e-commerce order state, payment state, and fulfillment state.
The database design separates these concerns, and API documentation must preserve that separation.

## Required reading

- [[00-start-here/README|Start Here]]
- [[01-product/project-scope|Production Project Scope]]
- [[01-product/production-module-catalog|Production Module Catalog]]
- [[02-architecture/system-overview|System Overview]]
- [[02-architecture/tenancy-architecture|Tenancy Architecture]]
- [[02-architecture/role-permission-capability-model|Role Permission Capability Model]]
- [[03-data/database-overview|Database Overview]]
- [[03-data/tenant-consistency-rules|Tenant Consistency Rules]]
- [[09-security-and-compliance/authorization-model|Authorization Model]]
- [[09-security-and-compliance/audit-requirements|Audit Requirements]]


## Related tables

| Table | Purpose |
|---|---|
| `orders` | E-Commerce order header with order/payment/fulfillment status columns. |
| `order_items` | Order line items. |
| `order_addresses` | Immutable address snapshots. |
| `order_status_transitions` | Allowed status transitions. |
| `order_status_history` | Status change history. |
| `payments` and allocations | Payment state and order payment allocation. |
| `deliveries` and delivery items | Fulfillment/delivery state. |
| `stock_reservations` | Online order reservation holds. |

## Status separation rule

Do not mix these statuses into one field:

| Status type | Meaning |
|---|---|
| Order status | Commercial order lifecycle. |
| Payment status | Payment authorization/capture/refund lifecycle. |
| Fulfillment status | Picking, pickup, delivery, collection lifecycle. |

The API must validate each status type using `order_status_transitions` where applicable.

## Order status examples from database design

- draft;
- pending_payment;
- confirmed;
- processing;
- completed;
- cancelled;
- partially_refunded;
- refunded.

## Payment status examples from database design

- pending;
- authorized;
- captured;
- failed;
- cancelled;
- voided;
- partially_refunded;
- refunded;
- expired.

## Fulfillment status examples from database design

- unfulfilled;
- reserved;
- picking;
- ready_for_pickup;
- out_for_delivery;
- delivered;
- collected;
- failed;
- cancelled.

## Transition API behavior

A transition request must include or resolve:

| Required concept | Reason |
|---|---|
| `status_type` | order, payment, or fulfillment. |
| current status | Prevent stale updates. |
| target status | Validate against transition table. |
| actor | Permission and audit. |
| reason | Required for cancellation/refund/failure where applicable. |
| related side effects | Reservation, delivery, payment, reporting. |

## Transition side effects

| Transition area | Possible side effects |
|---|---|
| Payment captured | Order may move out of pending payment depending on rules. |
| Stock reserved | Fulfillment status may become reserved. |
| Ready for pickup | Customer/order tracking update. |
| Delivered/collected | Fulfillment completion. |
| Cancelled | Reservation release, payment void/refund handling. |
| Refunded | Payment/order status update and report impact. |

## Invalid transition handling

If a transition is not allowed, return an `INVALID_STATUS_TRANSITION` error.
Do not update the order and then try to repair history later.

## History rule

Important status changes must create `order_status_history` rows with:

- tenant;
- order;
- status type;
- from status;
- to status;
- actor;
- timestamp;
- reason where applicable.

## API checklist

- [ ] Status type is explicit.
- [ ] Current state is checked.
- [ ] Target state is allowed.
- [ ] Actor has permission.
- [ ] Feature access is checked if feature-controlled.
- [ ] Side effects are handled in transaction/service boundary.
- [ ] History row is written.
- [ ] Error response is clear for invalid transition.
