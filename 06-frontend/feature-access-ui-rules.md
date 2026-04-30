---
title: Feature Access UI Rules
folder: 06-frontend
status: production-ready
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_frontend_architecture: Frontend archi V1.txt
source_backend_architecture: Back end Architecture final.txt
last_reviewed: 2026-04-30
tags: [frontend, react, typescript, tailwind, pos, ecommerce, unified-commerce]
---

# Feature Access UI Rules

This document defines how the frontend should reflect role, permission and feature availability.
It is aligned with the database model for platform features, tenant entitlements and role feature assignments.

## Required reading

- [[00-start-here/README]] — entry point for the Unified Commerce 2nd Brain.
- [[01-product/project-scope]] — production scope and system boundary.
- [[01-product/production-module-catalog]] — product-level module map.
- [[02-architecture/frontend-architecture]] — source architecture for React frontend structure.
- [[02-architecture/offline-first-architecture]] — offline-first POS design.
- [[03-data/database-overview]] — source data model and table ownership.
- [[04-api/api-overview]] — API design and `/api/v1` rules.
- [[05-backend/backend-overview]] — backend authority, service, repository and transaction rules.
- [[09-security-and-compliance/authorization-model]] — RBAC, feature access and tenant isolation rules.

## Core principle

Frontend feature access is a usability layer, not a security boundary.
The backend must remain the final authority.

## Source model

The database design includes:

| Table | Meaning |
|---|---|
| `platform_features` | Platform-owned feature catalog. |
| `tenant_feature_entitlements` | Features enabled for a tenant by platform admin. |
| `role_feature_assignments` | Features assigned to tenant roles. |
| `permissions` | Platform permission catalog. |
| `role_permissions` | Permissions assigned to tenant roles. |
| `tenant_user_roles` | Tenant-level role assignment. |
| `outlet_user_roles` | Outlet-level role assignment. |

Frontend must consume access context from backend APIs, not recreate these rules locally.

## UI behavior matrix

| Backend access result | Frontend behavior |
|---|---|
| Feature enabled + permission allowed | Show and enable action. |
| Feature enabled + permission denied | Hide or disable action with restricted explanation. |
| Feature disabled for tenant | Hide feature entry point or show unavailable state. |
| Feature enabled but outlet/user scope blocks action | Show scoped access message. |
| Backend rejects action | Show error from API contract and do not assume success. |

## Examples

| Feature | UI access behavior |
|---|---|
| POS billing | POS page accessible only if feature and role allow. |
| Refund | Refund button visible only to allowed roles; backend still validates. |
| Receipt reprint | Reprint action requires permission and audit-aware confirmation. |
| Stock adjustment | Inventory UI must hide adjustment action when not allowed. |
| Discount approval | Approval controls visible only to manager/authorized role. |
| Theme configuration | Tenant theme screen visible only to allowed admin role. |

## Feature context loading

Access context should be loaded early after login/session creation.
It should include enough data for:

- route visibility;
- navigation menu filtering;
- button/action availability;
- feature-disabled states;
- outlet-specific access;
- POS till/session screens.

Do not hard-code role names such as `admin` or `cashier` as final authority.
The project supports configurable tenant roles.

## Navigation rules

Navigation must be generated from feature/access context.

| Navigation area | Rule |
|---|---|
| POS navigation | Show only active POS functions for current outlet/session. |
| Admin menu | Show only allowed configuration/report modules. |
| Inventory menu | Show stock screens based on inventory permissions. |
| E-commerce ops menu | Show order/fulfillment actions based on feature access. |
| Reporting menu | Show only reports user may view. |

## Button and action rules

Sensitive action buttons should handle three states:

```text
visible + enabled
visible + disabled with reason
hidden when the feature/module is unavailable
```

For destructive or sensitive actions, disabled-with-reason is often better than invisible because it explains why the operator cannot proceed.

## Backend rejection rule

Even if the UI showed an action, backend may reject it because:

- permission changed;
- feature disabled;
- tenant suspended;
- till session closed;
- outlet context invalid;
- stock/payment/order status changed.

Frontend must show a clear message and refresh related state.

## Offline access rule

Offline POS must not assume all features are available.
Only offline-safe actions defined in the offline rules may be performed while offline.
Feature access context should be cached only if it is safe and tied to the current tenant/outlet/session.

## Anti-patterns

Do not:

- hard-code access by role name;
- assume tenant admin always has all feature rights;
- let frontend-only checks protect refunds or stock adjustments;
- hide backend errors behind generic failure messages;
- show POS actions when no active till/session exists;
- permit cross-tenant data display because cached state was not reset.

## Checklist

- [ ] Feature access comes from backend/session context.
- [ ] UI handles enabled, disabled and forbidden states.
- [ ] Role names are not hard-coded as final authority.
- [ ] Sensitive actions are still backend-validated.
- [ ] Offline mode does not expand feature access.
- [ ] Access changes clear or refresh related UI state.
