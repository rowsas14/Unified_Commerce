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
# Tenant Context API Rules

Tenant context is mandatory for the Unified Commerce platform because the system is multi-tenant.
APIs must never mix tenant data across users, outlets, devices, customers, stock, sales, orders, payments, reports, or settings.

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


## Tenant context sources

| Caller type | Tenant context source | Rule |
|---|---|---|
| Tenant staff | Auth/session claims | Do not trust body-only `tenant_id`. |
| Outlet staff/cashier | Auth + outlet assignment + POS device/session | Validate outlet and session context. |
| Customer | Storefront tenant context + customer account | Customer data is tenant-scoped. |
| Platform admin | Explicit platform-admin tenant selection | Only through platform-authorized routes. |
| Offline POS terminal | Registered device + sync batch payload | Device/outlet/tenant must match server records. |

## Tenant-owned data

Tenant context applies to records such as:

- users and roles;
- outlets and outlet addresses;
- tenant settings, feature flags, and themes;
- categories, products, variants, prices, tax classes;
- inventory balances, stock movements, reservations;
- tills, POS devices, till sessions, cash movements;
- sales, sale lines, receipts;
- customers, carts, orders, addresses;
- payments, allocations, refunds;
- discounts, coupons, returns, exchanges;
- deliveries and tracking;
- offline sync records;
- reporting summaries and audit logs.

## API context hierarchy

```text
Platform route
  → platform user identity
  → selected tenant where applicable

Tenant route
  → tenant user identity
  → tenant context from auth/session

POS route
  → tenant user identity
  → outlet assignment
  → registered POS device
  → active till/session where required

Customer route
  → storefront tenant context
  → customer identity or guest token
```

## Tenant ID handling

| Situation | Allowed? | Notes |
|---|---|---|
| Tenant staff sends `tenantId` in body to override context | No | Must use authenticated tenant context. |
| Platform admin route selects tenant in route | Yes | Only for platform-authorized routes. |
| POS sync item includes tenant ID | Conditional | Must match registered device tenant and sync batch tenant. |
| Customer route includes tenant context through storefront | Yes | Must match tenant-scoped customer/account. |
| Reporting filter includes outlet ID | Yes | Must belong to current tenant and user access scope. |

## Outlet context rules

Outlet context is required for:

- POS sales;
- POS device and till session operations;
- cash movements and cash counts;
- inventory balances and stock movements;
- stock transfers and stocktakes;
- pickup/fulfillment where outlet is assigned;
- outlet-level reporting.

APIs must validate that `outlet_id` belongs to the same tenant and that the user/device is allowed to operate there.

## Device and session context rules

POS APIs must validate:

| Context | Required for |
|---|---|
| `pos_device` | POS terminal, offline sync, source device tracking. |
| `till` | Session and cash drawer context. |
| `till_session` | Sales, payments, cash movement, close session. |
| `business_date` | Till sessions, sales, reports. |
| `source_device_id` | Offline dedupe and sync traceability. |

## Cross-tenant FK validation

Every API that accepts an ID referencing a tenant-owned table must validate same-tenant ownership.

Examples:

| Submitted ID | Validate against |
|---|---|
| `variant_id` | Product variant belongs to tenant. |
| `outlet_id` | Outlet belongs to tenant. |
| `customer_id` | Customer belongs to tenant. |
| `price_list_id` | Price list belongs to tenant. |
| `payment_id` | Payment belongs to tenant. |
| `return_id` | Return belongs to tenant. |
| `delivery_id` | Delivery belongs to tenant. |
| `sync_batch_id` | Sync batch belongs to tenant and device. |

## Platform user rule

Platform users are not tenant staff.
They operate through platform routes and may manage tenant lifecycle and entitlements.
They must not be treated as tenant users in normal POS/e-commerce workflows.

## Tenant context checklist

- [ ] Tenant is resolved before business logic runs.
- [ ] Body `tenantId` is not trusted for tenant staff operations.
- [ ] Outlet belongs to tenant.
- [ ] Device belongs to tenant and outlet where needed.
- [ ] Till belongs to tenant and outlet where needed.
- [ ] User has access to outlet where needed.
- [ ] Customer belongs to same tenant.
- [ ] All FK IDs are tenant-validated.
- [ ] Query results are tenant-filtered.
- [ ] Audit records include tenant context where applicable.
