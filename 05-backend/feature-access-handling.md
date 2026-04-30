---
title: Feature Access Handling
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

# Feature Access Handling

Feature access is a production requirement for this multi-tenant SaaS platform.
The database design includes a platform feature catalog, tenant feature entitlements, role feature assignments, runtime feature flags, and tenant settings.
The backend must enforce these rules; frontend visibility is not security.

## Feature access data model

| Table | Purpose |
|---|---|
| `platform_features` | Platform-owned feature catalog. |
| `tenant_feature_entitlements` | Platform admin enables or disables features for a tenant. |
| `role_feature_assignments` | Assigns tenant-enabled features to tenant roles. |
| `feature_flags` | Runtime enabled/configured state by tenant, outlet, or user scope. |
| `tenant_settings` | Tenant/outlet/channel settings for feature behavior. |
| `role_permissions` | Permission checks remain separate from feature assignment. |

## Access decision model

A backend action is allowed only when all required checks pass.

```text
Action request
  -> tenant exists and is active
  -> feature exists in platform feature catalog
  -> tenant entitlement allows feature
  -> role feature assignment allows feature where required
  -> runtime feature flag/config allows feature where required
  -> user permission allows action
  -> business validation passes
```

## Feature access vs permission

| Concept | Meaning |
|---|---|
| Feature entitlement | Platform says tenant may use this feature. |
| Role feature assignment | Tenant configuration says this role may access this feature. |
| Feature flag | Runtime configuration enables/disables or scopes behavior. |
| Permission | User is allowed to perform a specific action. |

A user may have a permission but still be blocked if the feature is not entitled or enabled.
A tenant may have a feature entitled but a user can still be blocked without permission.

## Backend service rule

Application services must call a feature/access decision service before protected operations.
The exact class name is an implementation detail, but the behavior must include tenant, role, feature, flag, and permission validation.

## Required context

| Context | Needed for |
|---|---|
| `tenant_id` | All tenant feature checks. |
| `user_id` | Staff role/permission checks. |
| `role_id` or resolved roles | Role feature assignment and permissions. |
| `outlet_id` | Outlet-scoped flags/settings and outlet role checks. |
| `feature_key` | Platform feature lookup. |
| `permission_code` | Action-level authorization. |
| `channel` | POS vs e-commerce channel behavior. |

## Scope priority

The uploaded scope requires tenant/outlet/channel/user configuration readiness.
Where settings overlap, backend must use documented priority from configuration docs.
Do not invent random priority in service code.
If not yet defined in the feature spec, stop and update documentation before implementation.

## Feature access examples

| Scenario | Required checks |
|---|---|
| Cashier opens POS checkout | Tenant POS feature entitlement, outlet role, POS feature assignment, till/session permission. |
| Manager approves discount | Discount feature entitlement, role feature assignment, discount approval permission, policy threshold. |
| Tenant admin edits theme | Theme/config feature access, tenant admin permission, audit. |
| POS offline sync | Offline POS entitlement/flag, registered device, tenant/outlet match. |
| Receipt reprint | Receipt feature access, reprint permission, print log/audit. |

## Failure handling

| Failure | Backend response behavior |
|---|---|
| Feature not in platform catalog | Treat as configuration error or feature unavailable. |
| Tenant entitlement disabled | Reject action as feature not enabled for tenant. |
| Runtime flag disabled | Reject or hide behavior according to API contract. |
| Role feature assignment denied | Reject action even if permission exists. |
| Permission missing | Reject with authorization error. |

## Audit rule

Changes to feature entitlements, role feature assignments, feature flags, tenant settings, and UI themes are sensitive configuration actions.
They should be auditable through `audit_logs` according to [[09-security-and-compliance/audit-requirements]].

## Implementation boundaries

- Do not hard-code feature access by fixed default role.
- Do not assume `Admin` can do everything without checking tenant boundaries.
- Do not use frontend route guards as final authorization.
- Do not duplicate feature names as free-text flags outside `platform_features`/`feature_flags`.
- Do not implement feature access only in controllers; Application services must enforce it.

## Checklist

- [ ] Feature exists in platform feature catalog.
- [ ] Tenant entitlement is enabled.
- [ ] Runtime feature flag/config allows the requested scope.
- [ ] User role has feature assignment where required.
- [ ] User has required permission.
- [ ] Outlet/channel scope is valid.
- [ ] Denied feature access returns safe API error.
- [ ] Configuration changes are audited.
