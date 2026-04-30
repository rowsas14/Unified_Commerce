---
title: Authentication and Authorization
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

# Authentication and Authorization

This file defines backend authentication and authorization rules for platform users, tenant staff, outlet staff, and customers.
The database design separates platform users, tenant users, customer profiles, customer auth accounts, customer auth identities, roles, permissions, feature entitlements, role feature assignments, and OTP verification records.

## Identity types

| Identity type | Database area | Backend meaning |
|---|---|---|
| Platform user | `platform_users` | Platform-side administrators who create tenants and manage entitlements. |
| Tenant staff user | `users` | Tenant-bound staff/admin users. |
| Outlet staff user | `users` + `outlet_user_roles` | Tenant user with outlet-scoped role access. |
| Customer | `customers` | Tenant-scoped customer profile. |
| Customer auth account | `customer_auth_accounts`, `customer_auth_identities` | Online login identity for customer channel. |

## Authentication responsibilities

| Area | Backend responsibility |
|---|---|
| Login | Validate credentials and account status. |
| Logout | End/clear session/token context according to auth implementation. |
| Staff identity | Load tenant, user, role, outlet role, permission, and feature context. |
| Customer identity | Load tenant-scoped customer auth account and customer profile. |
| OTP | Use hashed OTP only; validate attempt/resend/block state. |
| Last login | Update last successful login where table supports it. |

## Authorization sources

| Source | Purpose |
|---|---|
| `roles` | Tenant-owned role definitions. |
| `permissions` | Platform-owned permission catalog. |
| `role_permissions` | Maps tenant roles to permissions. |
| `tenant_user_roles` | Tenant-scope role assignment. |
| `outlet_user_roles` | Outlet-scope role assignment. |
| `platform_features` | Platform feature catalog. |
| `tenant_feature_entitlements` | Platform admin enables feature for tenant. |
| `role_feature_assignments` | Tenant-side feature assignment to roles. |
| `feature_flags` | Runtime feature configuration by tenant/outlet/user. |

## Authorization check order

```text
Authenticate identity
  -> validate tenant context
  -> validate outlet context where required
  -> validate account status
  -> validate permission
  -> validate feature entitlement
  -> validate role feature assignment
  -> validate runtime feature flag/config
  -> execute action
```

## Platform user rules

Platform users are not tenant staff.
Do not store platform admin flags in tenant `users`.
Platform operations may have `tenant_id` null in audit logs when the action is platform-level.
Tenant-specific platform support actions should record tenant context where applicable.

## Tenant staff rules

Tenant staff users must:

- belong to one tenant through `users.tenant_id`;
- receive roles through tenant or outlet role tables;
- access only tenant-owned records;
- use outlet access only where assignment exists;
- pass permission checks for sensitive actions.

## Customer auth rules

Customers are tenant-scoped, not global.
Same email can exist under different tenants as separate customer identities.
Customer login must not grant tenant staff/admin permissions.
Customer account actions must remain within the tenant/storefront context.

## Sensitive permissions

The uploaded scope calls out sensitive operations such as refund, void, stock adjustment, discount approval, receipt reprint, configuration changes, and offline conflict resolution.
Backend must separately authorize these operations.

| Sensitive action | Required backend check |
|---|---|
| Refund approval/payment | Permission + payment/refund validation + audit. |
| Sale void/cancel | Permission + sale status validation + reason + audit. |
| Discount approval | Permission + threshold validation + audit. |
| Stock adjustment | Permission + reason + stock movement rules + audit. |
| Receipt reprint | Permission + print log + audit. |
| Offline conflict resolution | Permission + conflict state validation + sync audit. |
| Feature configuration | Tenant entitlement + admin permission + audit. |

## Outlet authorization

Outlet-level actions must validate:

- user belongs to tenant;
- outlet belongs to tenant;
- user has active outlet role if outlet-scoped operation;
- POS device/till session belongs to same outlet.

## Backend vs frontend authority

Frontend guards are useful for user experience, but backend is final authority.
Even if the frontend hides a button, the backend must reject unauthorized requests.

## Related documents

- [[09-security-and-compliance/authentication-model]]
- [[09-security-and-compliance/authorization-model]]
- [[05-backend/feature-access-handling]]
- [[04-api/auth-and-authorization]]
- [[03-data/entities/identity-access-entities]]

## Checklist

- [ ] Identity type is known.
- [ ] Account status is active.
- [ ] Tenant context is valid.
- [ ] Outlet context is valid when required.
- [ ] Permission is checked.
- [ ] Feature access is checked.
- [ ] Sensitive action reason/audit is captured where required.
- [ ] Customer actions cannot access staff/admin APIs.
