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
# Feature Access API Rules

Feature access controls which production capabilities are available to a tenant and which tenant roles can use them.
APIs must enforce feature access on the backend.

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


## Feature access data model

| Table | Purpose |
|---|---|
| `platform_features` | Platform-owned catalog of available product features. |
| `tenant_feature_entitlements` | Platform admin enables features for a tenant. |
| `role_feature_assignments` | Tenant role is allowed to use an enabled feature. |
| `feature_flags` | Tenant runtime configuration by tenant/outlet/user scope. |
| `permissions` | Platform-owned action permission catalog. |
| `role_permissions` | Tenant role to permission mapping. |

## Required check sequence

```text
1. Resolve tenant context
2. Resolve authenticated actor
3. Verify platform feature exists and is active
4. Verify tenant entitlement is enabled
5. Verify runtime feature flag/config allows action
6. Verify role feature assignment allows feature
7. Verify permission allows specific action
8. Verify outlet/device/session context where required
9. Execute application service
```

## Why feature access is separate from permission

| Control | Answers |
|---|---|
| Feature entitlement | Is this feature available to this tenant at all? |
| Feature flag/config | Is this feature enabled/configured for this tenant/outlet/user now? |
| Role feature assignment | Is this role allowed to use this feature? |
| Permission | Can this actor perform this specific action? |

A user may have a permission code, but the action must still be blocked if the feature is not enabled for the tenant.

## Scope examples

| Feature area | API implication |
|---|---|
| POS billing | Sales APIs must check POS billing feature access. |
| Offline POS | Sync APIs must check offline feature availability and device status. |
| Returns/exchanges | Return/exchange APIs must check feature and permissions. |
| Loyalty | Loyalty APIs must check feature availability before use. |
| Reviews/wishlist | Storefront APIs must respect tenant feature configuration. |
| Receipt templates | Admin APIs must check receipt configuration feature access. |
| Reporting | Report APIs must check reporting permissions and feature access. |

## Feature flag scope rules

| Scope | API behavior |
|---|---|
| Tenant | Applies to all users/outlets/channels under tenant. |
| Outlet | Applies only when request outlet matches flag outlet. |
| User | Applies only to the user context. |

If multiple scopes apply, the configuration priority must be defined in the relevant feature specification.

## API error behavior

| Failure | Error category |
|---|---|
| Feature not in platform catalog | Configuration error. |
| Feature inactive platform-wide | Feature disabled. |
| Tenant not entitled | Feature access denied. |
| Runtime flag disabled | Feature disabled. |
| Role feature denied | Forbidden. |
| Permission missing | Forbidden. |

## Backend implementation notes

- Do not duplicate feature checks in controllers manually in many places.
- Use a shared feature access service or policy abstraction in the backend application layer.
- Feature checks must run before state-changing business operations.
- Feature check results may be cached carefully, but configuration changes must invalidate cache.
- Audit feature assignment/configuration changes.

## Frontend integration notes

Frontend can hide disabled actions based on feature configuration.
However, API must still validate feature access because hidden UI is not security.

## Checklist

- [ ] API route is mapped to a platform feature where feature-controlled.
- [ ] Tenant entitlement check is documented.
- [ ] Feature flag/config scope is documented.
- [ ] Role feature assignment is checked.
- [ ] Permission code is checked.
- [ ] Error behavior is predictable.
- [ ] Feature configuration changes are auditable.
