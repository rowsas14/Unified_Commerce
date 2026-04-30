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
# Authentication and Authorization API Rules

APIs must support platform users, tenant staff users, outlet staff, and e-commerce customers without mixing identities or permissions.
Authorization must be enforced by the backend, not by frontend visibility alone.

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


## Identity types

| Identity | Source tables / area | API scope |
|---|---|---|
| Platform user | `platform_users` | Platform admin routes, tenant entitlement management, support operations. |
| Tenant staff user | `users` | Tenant admin, manager, cashier, inventory, reporting, e-commerce operations. |
| Outlet staff | `users` + `outlet_user_roles` | Outlet-scoped POS/inventory/session operations. |
| Customer | `customers` + auth account/identity tables | Storefront account, cart, order, wishlist, review. |
| Device | `pos_devices` | POS terminal and offline sync context, not a human identity. |

## Authentication rules

| API area | Authentication requirement |
|---|---|
| Platform routes | Platform user authentication. |
| Tenant admin routes | Tenant staff authentication. |
| POS routes | Tenant staff authentication plus device/session context where required. |
| Storefront cart/order routes | Customer auth or tenant-scoped guest token where allowed. |
| Customer account routes | Customer authentication. |
| Offline sync routes | Registered device plus authenticated/authorized sync context. |
| Public product listing | Tenant storefront context; public fields only. |

## Authorization model

Authorization combines:

```text
Tenant context
  + Feature entitlement
  + Runtime feature flag/configuration
  + Role feature assignment
  + Permission
  + Outlet/device/session context where required
```

See [[04-api/feature-access-api-rules]].

## Staff role/permission sources

| Data source | Purpose |
|---|---|
| `roles` | Tenant-owned role definitions. |
| `permissions` | Platform-owned permission catalog. |
| `role_permissions` | Role-to-permission mapping. |
| `tenant_user_roles` | Tenant-level role assignment. |
| `outlet_user_roles` | Outlet-level role assignment. |
| `role_feature_assignments` | Role access to enabled platform features. |

## Sensitive operation examples

These APIs must require explicit permission and audit support:

- refund approval/payment;
- void/cancel completed sale;
- receipt reprint;
- stock adjustment approval/posting;
- discount approval;
- cash variance approval;
- tenant feature assignment;
- POS device/till assignment;
- offline sync conflict resolution;
- role/permission changes;
- payment provider configuration changes.

## Customer authorization rules

Customer APIs must ensure:

- customer belongs to current tenant;
- customer sees only their own orders/carts/wishlist/reviews;
- guest token is tenant-scoped;
- customer account identity does not become global across tenants;
- customer cannot access staff/admin APIs.

## Platform admin rules

Platform admins may manage tenant lifecycle and entitlements through platform routes.
They are not automatically tenant cashiers, tenant managers, or tenant inventory users.
Platform support access must still be intentionally scoped and audited.

## Frontend guard limitation

The frontend architecture includes route guards such as `AuthGuard`, `RoleGuard`, and `TillSessionGuard`.
These guards improve UX but are not security boundaries.
Every backend API must repeat authorization checks.

## Authorization response behavior

| Condition | Response behavior |
|---|---|
| Not authenticated | Return authentication error. |
| Authenticated but tenant invalid | Return access denied or tenant context error. |
| Feature not entitled/enabled | Return feature disabled/access denied. |
| Permission missing | Return forbidden. |
| Outlet/device/session invalid | Return context-specific forbidden/validation error. |
| Customer tries another account's data | Return not found or forbidden according to security policy. |

## Checklist

- [ ] Caller identity type is known.
- [ ] Route is protected or intentionally public.
- [ ] Tenant context is resolved.
- [ ] Permission is checked.
- [ ] Feature access is checked where applicable.
- [ ] Outlet role is checked where outlet scope is required.
- [ ] POS device/session context is checked where required.
- [ ] Sensitive action is audited.
- [ ] Response does not reveal unauthorized sensitive data.
