---
title: Offline Sync Conflicts Feature Spec
owner: Offline Sync Module Owner
status: production-ready-feature-spec
last_reviewed: 2026-04-30
tags:
  - unified-commerce
  - feature-spec
  - offline-sync
  - offline-sync-conflicts
---

# Offline Sync Conflicts Feature Spec

## Module

[[07-modules/offline-sync/README|Offline Sync Module]]

## Purpose

Conflict record when offline sync cannot be accepted cleanly.

This feature must be implemented as part of the production Unified Commerce E-POS + E-Commerce SaaS system. It must stay aligned with the uploaded production scope, uploaded database design, Clean Architecture backend rules, React frontend rules, API rules, security rules, and current 2nd Brain structure.

## Required reading before implementation

| Area | Document |
|---|---|
| Module README | [[07-modules/offline-sync/README|Offline Sync Module]] |
| Product scope | [[01-product/project-scope|Project Scope]] |
| Database overview | [[03-data/database-overview|Database Overview]] |
| Entity relationship map | [[03-data/entity-relationship-map|Entity Relationship Map]] |
| API rules | [[04-api/api-overview|API Overview]] |
| Backend rules | [[05-backend/backend-overview|Backend Overview]] |
| Frontend rules | [[06-frontend/frontend-overview|Frontend Overview]] |
| Security rules | [[09-security-and-compliance/README|Security and Compliance]] |
| Feature history | [[07-modules/offline-sync/features/offline-sync-conflicts/feature-history|Offline Sync Conflicts History]] |

## Actors

- Cashier
- Backend Service
- Frontend App
- Outlet Manager
- Support User

## Scope

### In scope

- Implement the behavior described by this feature only.
- Use approved entities: `offline_sync_conflicts`, `offline_sync_items`.
- Validate tenant context for every tenant-owned operation.
- Validate role permission and feature access before creating, updating, approving, posting, cancelling, or exposing sensitive data.
- Follow API request/response/error/idempotency rules from the API folder.
- Follow backend Service Pattern and Repository Pattern only.
- Follow frontend React + TypeScript + Tailwind + TanStack Query + Zustand rules where UI is required.

### Out of scope

- Do not create undocumented database tables.
- Do not add new statuses, workflows, or permissions that are not already documented.
- Do not bypass module boundaries by writing unrelated data directly.
- Do not implement CQRS, mediator handlers, or undocumented architectural patterns.
- Do not treat frontend visibility as security.


## Related entities and relationships

| Table | PK | Important FK / attribute references | Purpose |
| --- | --- | --- | --- |
| `offline_sync_conflicts` | `id` | tenant_id -> tenants.id; sync_item_id; device_id | Conflict record. |
| `offline_sync_items` | `id` | tenant_id -> tenants.id; sync_batch_id; device_id; entity_type | Generic sync item queue. |

## Business rules

- The feature must operate inside the current tenant context.
- Foreign keys must belong to the same tenant where the referenced table is tenant-owned.
- Create/update/delete behavior must follow the uploaded database constraints and table rules.
- If the feature touches money, stock, payment, refund, discount, receipt, offline sync, or audit data, the backend must be the final authority.
- If this feature participates in a workflow, invalid status transitions must be blocked.
- If this feature uses a source-of-truth ledger or history table, existing rows must not be mutated as a substitute for proper reversal/history behavior.

## Validation rules

| Validation area | Required rule |
|---|---|
| Tenant context | Request tenant must match every tenant-owned FK. |
| Identity/access | User must have required role/permission and enabled feature access. |
| Required fields | Required database fields must be present before persistence. |
| Status | Status values must use documented database CHECK values or reference data. |
| FK consistency | Referenced parent rows must exist and belong to the same tenant where applicable. |
| Duplicate prevention | Use documented unique keys and idempotency rules where applicable. |
| Audit/history | Sensitive changes must write audit/history records where approved tables exist. |

## API documentation requirement

This feature must not assume final route names. Before implementation, update or verify:

- [[04-api/api-overview|API Overview]]
- [[04-api/endpoint-design|Endpoint Design]]
- [[04-api/module-endpoint-map|Module Endpoint Map]]
- [[04-api/error-contract|Error Contract]]
- [[04-api/tenant-context-api-rules|Tenant Context API Rules]]
- [[04-api/feature-access-api-rules|Feature Access API Rules]]

## Backend implementation notes

- Place API request/response contracts in the API layer according to backend/API documentation.
- Place orchestration logic in the Application service for this module.
- Keep domain rules in domain entities or domain services only when they are pure business rules.
- Use repositories for data access.
- Use Unit of Work or transaction boundary rules for multi-table changes.
- Use validators before persistence.
- Return controlled errors using the API error contract.

## Frontend implementation notes

- Keep feature UI aligned with the documented frontend folder structure.
- Use TanStack Query for server state and mutations.
- Use Zustand only for local workflow state such as POS cart/session/offline state.
- Show clear loading, empty, validation error, permission denied, feature disabled, and failure states.
- For POS-facing UI, keep controls fast, touchscreen-first, and operational.

## User-flow references

- [[08-user-flows/manager/offline-conflict-resolution|Offline Conflict Resolution]]

## Permission and feature access notes

- Permission must be checked by backend, not only by frontend.
- Feature must be enabled for the tenant where it belongs to a configurable platform feature.
- Role feature assignment must be respected where the feature is controlled by platform feature access.
- Manager/admin approval rules must be enforced for sensitive actions where the scope/database documents define approval fields.

## QA checklist

- [ ] Happy path works with valid tenant and permission context.
- [ ] Cross-tenant access is rejected.
- [ ] Missing required fields are rejected.
- [ ] Invalid FK references are rejected.
- [ ] Duplicate records are blocked by unique/index rules where applicable.
- [ ] Feature-disabled state is handled.
- [ ] Permission-denied state is handled.
- [ ] Audit/history behavior is verified where applicable.
- [ ] API error contract is followed.
- [ ] Frontend displays practical operational states.
- [ ] Regression impact on related modules is reviewed.

## Implementation checklist

- [ ] Read module README.
- [ ] Read this feature spec.
- [ ] Read feature history.
- [ ] Verify database entity references in `03-data`.
- [ ] Verify API documentation.
- [ ] Implement backend service/repository changes.
- [ ] Implement frontend changes if required.
- [ ] Add/update tests.
- [ ] Update feature history after changes.

## Notes for future updates

If the source scope or database design changes, update this file before code changes. Do not allow AI IDE tools to infer new tables, endpoints, permissions, or workflows from this document.
