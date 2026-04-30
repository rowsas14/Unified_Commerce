---
title: Backend Implementation Checklist
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

# Backend Implementation Checklist

Use this checklist before creating or changing backend code for any Unified Commerce feature.
It is designed for human developers and AI IDE tools.

## Required documents

- [ ] Read [[01-product/project-scope]].
- [ ] Read related module doc under `07-modules/`.
- [ ] Read related entity doc under [[03-data/README]].
- [ ] Read API rule under [[04-api/README]].
- [ ] Read security rule under [[09-security-and-compliance/README]].
- [ ] Read relevant backend rule in this folder.

## Architecture gate

- [ ] Uses Clean Architecture.
- [ ] Uses Service Pattern.
- [ ] Uses Repository Pattern.
- [ ] Uses Unit of Work for multi-table workflows.
- [ ] Does not use CQRS.
- [ ] Does not use Mediator/MediatR guidance.
- [ ] Controllers are thin.
- [ ] Domain stays pure.

## Tenant/access gate

- [ ] Tenant context is validated.
- [ ] Outlet context is validated where needed.
- [ ] Device/till/session context is validated for POS.
- [ ] Permission is checked.
- [ ] Feature entitlement is checked.
- [ ] Role feature assignment/feature flag is checked where needed.
- [ ] Customer access is not mixed with staff access.

## Data gate

- [ ] Tables used exist in uploaded database design.
- [ ] PK/FK relationships are respected.
- [ ] Tenant-owned queries include tenant filter.
- [ ] Unique/index rules are respected.
- [ ] No unsupported table/entity is invented.
- [ ] Source-of-truth tables remain authoritative.

## Workflow gate

- [ ] Status transition is valid.
- [ ] Financial totals are backend-calculated or validated.
- [ ] Stock movement rules are applied.
- [ ] Payment/refund limits are applied.
- [ ] Offline sync dedupe is applied where relevant.
- [ ] Audit/history/print/sync logs are created where required.

## API/backend contract gate

- [ ] Request model is defined.
- [ ] Response model is defined.
- [ ] DTOs are not domain entities.
- [ ] Error codes follow API error contract.
- [ ] Pagination/filtering/sorting follows API rules where lists are used.
- [ ] Idempotency is applied for duplicate-prone workflows.

## Testing gate

- [ ] Unit tests can cover service validation.
- [ ] Integration tests can cover repository/database behavior.
- [ ] Tenant isolation tests are defined.
- [ ] Permission/feature access tests are defined.
- [ ] Workflow status tests are defined.
- [ ] Offline conflict tests are defined where relevant.

## Final reviewer checklist

- [ ] No business logic hidden in controllers.
- [ ] No EF Core in Domain.
- [ ] No direct DbContext use from API controllers.
- [ ] No plain password/OTP/payment secret exposure.
- [ ] No cross-tenant query risk.
- [ ] No fake schema additions.
- [ ] No CQRS/Mediator pattern added.
