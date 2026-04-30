---
title: Tenant Consistency Rules
folder: 03-data
status: production-ready
owner: Architecture / Backend / Data
tags: [tenant, isolation, consistency]
---

# Tenant Consistency Rules

Tenant consistency protects the SaaS boundary. Every tenant-owned record must be impossible to mix with another tenant's data.

---

## Tenant consistency by relationship

| Relationship | Rule |
|---|---|
| `outlets.tenant_id` | Outlet belongs to one tenant only. |
| `users.tenant_id` | Staff user belongs to one tenant only. |
| `roles.tenant_id` | Role is tenant-owned. |
| `role_permissions.tenant_id` | Role must belong to same tenant. |
| `products.tenant_id` | Product belongs to one tenant. |
| `product_variants.tenant_id` | Variant must match product tenant. |
| `inventory_balances` | Tenant, outlet, and variant must align. |
| `sales` | Tenant, outlet, till session, device, customer must align. |
| `sale_lines` | Sale and variant must belong to same tenant. |
| `orders` | Tenant, cart, customer, fulfillment outlet must align. |
| `order_items` | Order and variant must belong to same tenant. |
| `payments` | Tenant, customer, outlet, till session, method, and device must align when populated. |
| `returns` | Source sale/order must belong to same tenant. |
| `receipts` | Source document and template must belong to same tenant. |
| `offline_sync_items` | Batch/device/tenant must match. |

---

## Enforcement levels

| Layer | Responsibility |
|---|---|
| Database | FK constraints, unique constraints, generated columns, indexes. |
| EF Core | Relationship mapping, query filters where safe, explicit FK mapping. |
| Application service | Tenant consistency validation across multiple FK paths. |
| API middleware | Resolve tenant context and reject cross-tenant requests. |
| Tests | Attempt cross-tenant access and verify rejection. |

---

## Checklist for any new FK

- [ ] Does the referenced table carry `tenant_id` or belong to a global reference catalog?
- [ ] If tenant-owned, does the service validate parent and child tenant match?
- [ ] Does the API resolve tenant from authenticated context instead of trusting body input?
- [ ] Is the FK optional only for a valid business reason?
- [ ] Are query filters safe, or is explicit tenant filtering required?
- [ ] Is there a test for cross-tenant rejection?

Related: [[schema-principles]], [[ef-core-implementation-notes]], [[../02-architecture/tenancy-model]].
