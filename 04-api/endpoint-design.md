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
# Endpoint Design Rules

Endpoint design must reflect the production Unified Commerce modules, not only table CRUD.
Some modules are master-data oriented, while others are workflow-oriented and require strict state validation.

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


> **API contract rule**  
> This folder defines production API design rules and route-family conventions.  
> A final endpoint contract must still be documented in a feature API spec using [[12-templates/api-spec-template]].  
> Do not implement undocumented endpoints only because a table exists.


## Base path

```http
/api/v1
```

Every route in this system must be designed under this prefix.

## Endpoint design principles

| Principle | Meaning |
|---|---|
| Module-first | Start from the module and feature, not from the table alone. |
| Resource-oriented | Use resources for stable data such as products, customers, orders, payments. |
| Workflow-aware | Use action/transition endpoints when business state must be validated. |
| Tenant-safe | Avoid accepting trusted tenant context only from request body. |
| Idempotent where needed | Duplicate-prone operations must define idempotency. |
| Auditable | Sensitive operations must trigger audit behavior. |
| Versioned | All production APIs use `/api/v1`. |

## HTTP method usage

| Method | Use for | Notes |
|---|---|---|
| `GET` | Read resources, lookup data, reports | Must not mutate state. |
| `POST` | Create records or run workflow actions | Use idempotency for unsafe duplicate-prone actions. |
| `PUT` | Replace full configuration or resource when valid | Avoid for complex business workflows. |
| `PATCH` | Partial update where state rules are simple | Prefer explicit workflow endpoint for critical transitions. |
| `DELETE` | Rare physical delete | Most business documents should be cancelled, archived, voided, or marked inactive. |

## Route naming patterns

| Pattern | Use when | Example style |
|---|---|---|
| Collection | Listing or creating resources | `/api/v1/catalog/products` |
| Single resource | Reading or updating one resource | `/api/v1/orders/{orderId}` |
| Child collection | Child belongs to parent | `/api/v1/orders/{orderId}/items` |
| Workflow action | Business action is not simple CRUD | `/api/v1/pos/sessions/{sessionId}/close` |
| Transition collection | Status transition needs history | `/api/v1/orders/{orderId}/status-transitions` |
| Report resource | Read-only report endpoint | `/api/v1/reports/daily-sales` |

## Workflow endpoint examples by rule

The following are design patterns, not a complete endpoint list.

| Workflow | Endpoint style | Required validations |
|---|---|---|
| Open till session | `POST /api/v1/pos/sessions` | User/outlet/till/device access, no active session conflict. |
| Close till session | `POST /api/v1/pos/sessions/{id}/close` | Counted cash, expected cash, variance, manager approval if needed. |
| Complete POS sale | Workflow under `/api/v1/pos/sales` | Active session, stock, pricing/tax, payments, receipt side effects. |
| Capture payment | Workflow under `/api/v1/payments` | Payment method, provider/manual reference, idempotency. |
| Approve refund | Workflow under `/api/v1/refunds` | Permission, original captured payment, refund limit. |
| Create return | Resource/workflow under `/api/v1/returns` | Original sale/order, return policy, eligible quantity. |
| Resolve offline conflict | Workflow under `/api/v1/offline-sync/conflicts` | Manager permission, conflict type, explicit resolution. |

## Request body rules

Do not trust these fields just because the client sends them:

- `tenant_id` for tenant staff requests;
- computed totals;
- payment status;
- stock availability;
- final tax amount;
- role/permission decisions;
- feature enabled state;
- order/payment/fulfillment target status without transition validation.

Accept user intent fields where appropriate:

- selected product/variant IDs;
- quantities;
- reason text;
- selected payment method;
- external payment reference;
- return reason;
- discount request reason/value;
- offline client IDs;
- filter/search values.

## Status-changing endpoint rule

Any endpoint that changes a status must document:

| Required item | Reason |
|---|---|
| Current status | To validate transition. |
| Target status | To control workflow. |
| Actor | To enforce permission and audit. |
| Required permission | To prevent unauthorized transitions. |
| Side effects | To update payment, stock, receipt, reporting, audit. |
| History table | To preserve transition history where defined. |

## Financial endpoint rule

Endpoints that touch money must document:

- currency;
- amount precision;
- calculation source;
- discount/tax relationship;
- payment allocation;
- refund limit;
- idempotency key requirement;
- reporting impact;
- audit behavior.

## Inventory endpoint rule

Inventory APIs must preserve ledger traceability.
Do not design APIs that directly change stock balances without a documented source document or stock movement rule.

## Endpoint review checklist

- [ ] Uses `/api/v1`.
- [ ] Belongs to a production module.
- [ ] Matches route naming conventions.
- [ ] Has auth and tenant context rules.
- [ ] Has permission and feature rules where needed.
- [ ] Has request and response shape.
- [ ] Has error cases.
- [ ] Has idempotency and concurrency rules when needed.
- [ ] Has database table references.
- [ ] Has audit and testing requirements.
