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
# Payment and Refund API Rules

Payment and refund APIs are high-risk because they affect money, sales completion, order processing, returns, exchanges, cash reconciliation, and reporting.
They must be explicit, idempotent, auditable, and tenant-safe.

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


## Related database tables

| Table | Purpose |
|---|---|
| `payment_method_types` | Global method reference values. |
| `tenant_payment_methods` | Tenant-enabled methods and non-secret config. |
| `payment_provider_configs` | Provider config with secret references, not raw secrets. |
| `payments` | Unified payment/outbound payment record. |
| `payment_transactions` | Gateway/provider event log. |
| `sale_payment_allocations` | Allocate payments to POS sales. |
| `order_payment_allocations` | Allocate payments to e-commerce orders. |
| `refunds` | Business refund header. |
| `return_refund_allocations` | Allocate refunds to returns. |
| `exchange_payment_allocations` | Additional exchange collection. |
| `exchange_refund_allocations` | Exchange difference refund allocation. |

## Payment API responsibilities

| Responsibility | Rule |
|---|---|
| Record payment | Store amount, method, status, direction, purpose, references. |
| Allocate payment | Link captured payment to sale/order/exchange difference. |
| Track provider event | Store gateway/provider transaction event where applicable. |
| Support manual methods | Cash/card/QR references may be recorded where configured. |
| Support split payment | Multiple payments may allocate to one sale/order. |
| Support offline payment | Client payment IDs and sync batch fields support offline dedupe. |

## Refund API responsibilities

| Responsibility | Rule |
|---|---|
| Validate original payment | Refund must reference original captured payment. |
| Limit amount | Total refunds cannot exceed original captured amount. |
| Support outbound payment | Refund payment may be recorded as outbound payment row. |
| Allocate refund | Link refund to return or exchange where applicable. |
| Require approval where configured | Refund approval is sensitive and auditable. |
| Follow original method rule | Use original payment method unless manager override is allowed by documented policy. |

## Payment status rules

Payment statuses must follow the database design values:

- pending;
- authorized;
- captured;
- failed;
- cancelled;
- voided;
- partially_refunded;
- refunded;
- expired.

Do not create arbitrary frontend-only payment statuses.

## Payment direction and purpose

| Field | Values from database design |
|---|---|
| `payment_direction` | inbound, outbound |
| `payment_purpose` | sale, order, refund, exchange_difference |

APIs must validate direction/purpose combinations.

## Idempotency requirements

Payment and refund endpoints must use idempotency when duplicate request risk exists.
Relevant safeguards include:

- `payments.idempotency_key`;
- `payments.client_payment_id`;
- tenant-scoped unique constraints;
- provider transaction IDs where applicable.

See [[04-api/idempotency-rules]].

## Security rules

Payment APIs must not expose or accept raw provider secrets.
Payment provider configs may store non-secret config and secret references only.
See [[09-security-and-compliance/payment-security-rules]].

## Manual versus gateway payment

| Type | API behavior |
|---|---|
| Cash | Record payment and reconcile with till session/cash reporting. |
| Manual card/QR | Store external/reference number where applicable. |
| Gateway payment | Record provider config, transaction event, status updates. |
| Offline cash | Allowed only under offline POS rules and synced later. |
| Offline card/QR | Must follow configured external terminal/manual confirmation rules; do not assume gateway works offline. |

## Checklist

- [ ] Payment method is tenant-enabled.
- [ ] Provider config belongs to tenant where used.
- [ ] Payment amount and currency are valid.
- [ ] Payment purpose and direction are valid.
- [ ] Allocation does not exceed captured amount.
- [ ] Refund does not exceed original captured amount.
- [ ] Idempotency key/client payment ID is handled.
- [ ] Sensitive operation is audited.
- [ ] Provider secrets are not exposed.
