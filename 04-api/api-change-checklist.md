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
# API Change Checklist

Use this checklist before adding, changing, or removing any `/api/v1` API behavior.
API changes can affect POS terminals, offline sync, admin portal, storefront, backend services, reports, and AI IDE code generation.

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


## Change classification

| Change type | Examples | Risk |
|---|---|---|
| Additive | New optional field, new endpoint, new filter | Usually low if documented. |
| Behavioral | Same route behaves differently | High; may break POS/offline clients. |
| Breaking | Required field removed/renamed, response shape changed | Requires versioning or migration plan. |
| Security | Auth/permission/feature rule changed | Must be reviewed carefully. |
| Financial | Payment/refund/tax/discount behavior changed | High risk. |
| Sync | Offline payload/status behavior changed | High risk for delayed clients. |

## Pre-change checklist

- [ ] The change maps to a production module in [[01-product/production-module-catalog]].
- [ ] The feature spec exists or will be updated.
- [ ] The data model supports the change in [[03-data/database-overview]].
- [ ] No unapproved entity/table is being invented.
- [ ] `/api/v1` compatibility impact is understood.
- [ ] Tenant isolation impact is reviewed.
- [ ] Authorization and feature access impact is reviewed.
- [ ] Frontend consumers are identified.
- [ ] Backend service/repository impact is identified.
- [ ] Tests and regression scenarios are identified.

## Documentation updates required

| If changed | Update these docs |
|---|---|
| Endpoint contract | API spec, feature spec, frontend API usage, tests. |
| Request/response shape | [[04-api/request-response-standard]], related feature spec. |
| Error behavior | [[04-api/error-contract]]. |
| Auth/permission | [[04-api/auth-and-authorization]], security docs. |
| Feature access | [[04-api/feature-access-api-rules]]. |
| Tenant/outlet/device context | [[04-api/tenant-context-api-rules]], [[04-api/device-session-api-rules]]. |
| Offline sync | [[04-api/offline-sync-api-rules]], offline data docs, tests. |
| Payment/refund | [[04-api/payment-refund-api-rules]], security, tests. |
| Order workflow | [[04-api/order-workflow-api-rules]], user flows, tests. |

## Compatibility checklist

- [ ] Existing POS terminals can still use the route.
- [ ] Offline sync clients with delayed payloads remain compatible.
- [ ] Storefront clients are not broken.
- [ ] Admin pages using TanStack Query are updated.
- [ ] Response envelope remains stable.
- [ ] Error codes remain stable or are documented.
- [ ] Idempotency behavior remains stable.
- [ ] Pagination/filter behavior remains stable.

## Security checklist

- [ ] Auth requirement is correct.
- [ ] Tenant context is validated.
- [ ] Outlet/device/session context is validated where needed.
- [ ] Permission is checked.
- [ ] Feature access is checked.
- [ ] Sensitive fields are not exposed.
- [ ] Audit behavior is defined for sensitive operations.
- [ ] Payment/provider secrets are not exposed.

## Financial and inventory checklist

- [ ] Price/tax/discount totals are backend-validated.
- [ ] Payment/refund amount limits are enforced.
- [ ] Stock changes are ledger-backed.
- [ ] Coupon usage limits are concurrency-safe.
- [ ] Document sequence allocation is safe.
- [ ] Reports/read models are updated if needed.

## Release checklist

- [ ] API docs updated.
- [ ] Backend implementation checklist updated.
- [ ] Frontend integration notes updated.
- [ ] Automated tests planned/updated.
- [ ] Manual QA scenarios documented.
- [ ] Migration/seed impact reviewed.
- [ ] Rollback or compatibility plan exists for risky change.
- [ ] Changelog/feature history updated.
