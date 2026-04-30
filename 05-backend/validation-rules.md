---
title: Validation Rules
folder: 05-backend
status: production-ready
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_backend_architecture: Back end Architecture final.txt
source_frontend_architecture: Frontend archi V1.txt
stack: .NET Web API, Clean Architecture, PostgreSQL, EF Core
patterns: Service Pattern, Repository Pattern, Unit of Work
cqrs: not-used
---

# Validation Rules

Validation protects tenant data, financial totals, inventory integrity, workflow states, and offline sync safety.
This file defines backend validation responsibilities for the production Unified Commerce system.

Validation must be layered. API validation catches malformed input, Application validation checks use-case rules, Domain validation handles pure business rules, and database constraints remain the final structural guard.

## Validation layers

| Layer | Validation responsibility |
|---|---|
| API | Required request shape, route/body consistency, basic type binding. |
| Application | Tenant, permission, feature, workflow, state, and cross-table validation. |
| Domain | Pure business invariants that do not depend on persistence. |
| Database | PK/FK, unique indexes, check constraints, generated columns, relational integrity. |

## Uploaded architecture alignment

The backend architecture includes validators such as:

- `ProductValidator.cs`
- `OrderValidator.cs`
- `PaymentService` validation behavior
- `CustomerValidator.cs`
- `InventoryValidator.cs`

These validators belong in the Application layer unless they are simple API model validation attributes.

## Global validation order

```text
API receives request
  -> basic request validation
  -> authentication/session validation
  -> tenant/outlet/device/channel context validation
  -> permission and feature validation
  -> application validator
  -> domain rule validation
  -> database constraint safety
```

## Tenant validation

| Rule | Required behavior |
|---|---|
| Tenant-owned data | Must be queried with `tenant_id` or tenant-scoped parent. |
| Outlet-owned workflows | Validate outlet belongs to tenant. |
| User role assignment | Validate user, role, outlet belong to same tenant. |
| Customer records | Same email may exist across tenants; do not merge globally. |
| Device context | POS device outlet must match till/session outlet. |

## Feature access validation

Before performing protected actions, backend must validate:

1. Tenant has platform feature entitlement.
2. Runtime feature flag/config allows the operation where applicable.
3. User role has the feature assignment where applicable.
4. User has required permission.
5. Outlet or tenant scope allows the action.

Read [[05-backend/feature-access-handling]].

## POS validation

| POS area | Validation rule |
|---|---|
| Sale completion | Active tenant, active outlet, valid POS device, open till session where required. |
| Sale lines | Active/sellable variant, quantity greater than zero, correct tenant. |
| Discounts | Valid scope, type, approval status, threshold rule. |
| Payments | Payable balance, payment status, method enabled, split totals. |
| Receipt | Exactly one source document where receipt is generated. |
| Void/cancel | Permission, status, reason, audit required. |

## Inventory validation

| Inventory area | Validation rule |
|---|---|
| Stock movement | Quantity positive; movement type determines direction. |
| Balance | Tenant, outlet, variant must match. |
| Reservation | Active order item, reserved quantity positive, expiry/status valid. |
| Transfer | Source and destination outlet must differ. |
| Stocktake | Posted stocktake creates gain/loss movements only through service. |
| Damage | Damaged return must not automatically become sellable stock. |

## Payment and refund validation

| Area | Validation rule |
|---|---|
| Idempotency | Tenant-scoped idempotency key must prevent duplicate payment/order/sync effects. |
| Payment allocation | Allocated total must not exceed captured amount. |
| Refund | Total refunds cannot exceed original captured amount. |
| Refund payment | Outbound refund payment must have purpose `refund`. |
| Provider config | Gateway-backed method must use tenant-owned provider config. |
| Offline payment | Must use client payment ID dedupe when offline-created. |

## Order and fulfillment validation

| Area | Validation rule |
|---|---|
| Cart conversion | Cart must be active and not expired. |
| Order placement | Customer/guest, stock, price, tax, address snapshots must be valid. |
| Status transition | Must match allowed transition rules/history. |
| Fulfillment | Pickup and delivery statuses must not be mixed incorrectly. |
| Delivery items | Quantities must map to order items and tenant-owned variants. |

## Returns and exchanges validation

| Area | Validation rule |
|---|---|
| Return source | Exactly one source sale or source order. |
| Return line source | Exactly one source sale line or order item. |
| Quantity | Returned quantity cannot exceed eligible original quantity. |
| Policy | Return policy, window, non-returnable status, and manager override must be validated. |
| Exchange | Source return required; difference direction must match totals. |
| Allocation | Refund/payment allocation must match difference behavior. |

## Offline sync validation

| Area | Validation rule |
|---|---|
| Batch | Device, outlet, tenant must match. |
| Item | Unique tenant + device + entity type + client entity ID. |
| Typed queues | Sale/payment typed queues are staging only, not source of truth. |
| Conflict | Stock mismatch, duplicate, validation failed, closed session must create conflict record. |
| Sync audit | Sync lifecycle events must be logged. |

## Validation checklist

- [ ] Request shape is valid.
- [ ] Tenant context is valid.
- [ ] Outlet/device/session context is valid where needed.
- [ ] Permission is checked.
- [ ] Feature entitlement/flag is checked.
- [ ] Entity belongs to tenant.
- [ ] Status transition is valid.
- [ ] Financial totals are validated server-side.
- [ ] Inventory movement rules are valid.
- [ ] Offline dedupe is enforced.
- [ ] Database constraints support the rule.
