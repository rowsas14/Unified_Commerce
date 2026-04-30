---
title: Backend Implement Rule
owner: Architecture + AI IDE
status: production-ready
last_reviewed: 2026-04-30
tags: [ai-ide, backend, implementation, caching]
source: Unified Commerce production scope + database design + backend architecture
---

# Backend Implement Rule

## Purpose

This file defines how AI IDE tools and backend developers must implement backend features for the Unified Commerce E-POS + E-Commerce SaaS system.

The backend uses Clean Architecture with Service Pattern and Repository Pattern.

Do not use CQRS.

Do not use Mediator-based implementation guidance.

Read with:

- [[05-backend/backend-overview]]
- [[05-backend/backend-folder-structure]]
- [[05-backend/clean-architecture-rules]]
- [[05-backend/caching-strategy]]
- [[03-data/database-overview]]
- [[04-api/README]]

---

## Required implementation sequence

Before writing backend code:

1. Read the related product/module/feature documentation.
2. Read related entity docs in [[03-data/README]].
3. Read related API rules in [[04-api/README]].
4. Read backend rules in [[05-backend/README]].
5. Confirm tenant isolation and feature access behavior.
6. Confirm transaction boundary.
7. Confirm validation rules.
8. Implement service, repository, DTO, mapping and validation.
9. Add tests.
10. Update feature history.

---

## Required layer rules

| Layer | Allowed responsibility |
|---|---|
| API | Controllers, requests, responses, filters, middleware. |
| Application | Services, DTOs, validators, interfaces, orchestration. |
| Domain | Entities, value objects, pure business rules/domain services. |
| Infrastructure | EF Core, repositories, external integrations, Unit of Work. |

Do not place business rules in controllers.

Do not access EF Core directly from API controllers.

---

## Service Pattern rule

Application services coordinate use cases such as:

- Product creation/update.
- POS sale completion.
- Payment recording.
- Refund processing.
- Order placement.
- Stock movement posting.
- Return/exchange completion.
- Offline sync acceptance.

Services must validate tenant, outlet, feature access and permissions before executing sensitive actions.

---

## Repository Pattern rule

Repositories handle persistence concerns.

They must:

- Apply tenant filters.
- Use indexes correctly.
- Support transaction boundaries through Unit of Work.
- Avoid business decisions that belong in services/domain logic.
- Avoid returning cross-tenant data.

---

## Caching implementation rule

Before implementing backend caching, read:

- [[05-backend/caching-strategy]]
- [[03-data/indexing-strategy]]
- [[03-data/schema-principles]]
- [[14-ai-ide-rules/database-alignment-rule]]

Rules:

- Do not create cache tables.
- Do not create cache repositories.
- Do not add `backend_cache`, `query_cache`, `product_cache`, or `tenant_cache` tables.
- Use safe short-lived application-level cache only where documented.
- Include `tenant_id` in cache keys for tenant-owned data.
- Include outlet/channel/user/role context where needed.
- Revalidate PostgreSQL source tables before critical writes.

---

## Data that may use backend cache

Only safe read-heavy data may use backend cache:

- Tenant settings.
- Feature flags.
- UI themes.
- Permission/feature access context with short TTL.
- Product lookup/search data with short TTL.
- Category/brand/attribute lists.
- Price/tax setup with careful invalidation.
- Payment method configuration without secrets.
- Delivery methods/zones/rates.

---

## Data that must not use cache as truth

Do not use cache as final authority for:

- Sales.
- Orders.
- Payments.
- Refunds.
- Stock balances.
- Stock movements.
- Stock reservations.
- Till sessions.
- Cash movements.
- Coupon redemptions.
- OTP verification.
- Document sequences.
- Offline sync duplicates/conflicts.
- Audit logs.

---

## Transaction boundary reminders

Critical workflows must be transaction-safe:

- POS sale + sale lines + payment allocation + stock movement + receipt.
- Order placement + reservation + payment state.
- Refund + payment update + return allocation.
- Exchange + return + payment/refund difference + stock movement.
- Offline sync acceptance + source records + sync item update.
- Cash session close + cash count/variance.

Do not split critical business writes across unrelated service calls without transaction control.

---

## Validation reminders

Backend validation must cover:

- Tenant context.
- Outlet/device/session context.
- Role/permission/feature access.
- Status transition.
- FK tenant consistency.
- Idempotency.
- Stock availability/conflict.
- Payment/refund limits.
- Coupon/discount limits.
- Offline sync duplicate/conflict state.

---

## AI IDE forbidden output

Do not generate:

- CQRS handlers.
- Mediator pipelines.
- Controller business logic.
- Cross-tenant queries.
- Cache database tables.
- Cache repositories.
- Unapproved entities.
- Payment/stock/refund decisions from cache only.

---

## Backend implementation checklist

- [ ] Read feature spec.
- [ ] Read related entity docs.
- [ ] Read API rule docs.
- [ ] Confirm transaction boundary.
- [ ] Confirm tenant isolation.
- [ ] Confirm feature access and permission checks.
- [ ] Confirm cache is not source of truth.
- [ ] Implement service and repository correctly.
- [ ] Add validation.
- [ ] Add tests.
- [ ] Update feature history.

---

## Related files

- [[05-backend/caching-strategy]]
- [[03-data/indexing-strategy]]
- [[04-api/idempotency-rules]]
- [[04-api/concurrency-rules]]
- [[14-ai-ide-rules/database-alignment-rule]]
