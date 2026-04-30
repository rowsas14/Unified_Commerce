---
title: Coding Standard Gate Rule
owner: Engineering Leads
status: production-ready
last_reviewed: 2026-04-30
tags: [ai-ide, coding-standards, quality]
---

# Coding Standard Gate Rule

## Purpose

This rule tells AI IDE tools the coding standards that must be respected before generating or changing code.

It applies to backend, frontend, API contracts, tests, and documentation-linked implementation work.


## Core references

| Area | Required document |
|---|---|
| Project entry | [[00-start-here/README]] |
| Documentation rules | [[00-start-here/documentation-rules]] |
| Product scope | [[01-product/project-scope]] |
| Product modules | [[01-product/production-module-catalog]] |
| System overview | [[02-architecture/system-overview]] |
| Architecture principles | [[02-architecture/architecture-principles]] |
| Data model | [[03-data/database-overview]] |
| Entity relationships | [[03-data/entity-relationship-map]] |
| API rules | [[04-api/README]] |
| Backend rules | [[05-backend/README]] |
| Frontend rules | [[06-frontend/README]] |
| Security rules | [[09-security-and-compliance/README]] |
| Templates | [[12-templates/README]] |


## Global coding principles

| Principle | Rule |
|---|---|
| Clarity | Prefer readable, explicit code over clever code. |
| Boundaries | Keep logic in the correct layer. |
| Safety | Do not bypass tenant, security, payment, stock, or offline rules. |
| Consistency | Use project naming and folder conventions. |
| Minimal change | Avoid unrelated refactors. |
| Traceability | Update docs/tests when behavior changes. |

## Backend coding standard

| Area | Rule |
|---|---|
| Architecture | Clean Architecture only. |
| Pattern | Service Pattern + Repository Pattern. |
| Controllers | Thin HTTP boundary. |
| Services | Use-case orchestration. |
| Repositories | Persistence only. |
| DTOs | Do not expose EF entities directly. |
| Validation | Validate request and business state before persistence. |
| Transactions | Use Unit of Work for multi-table changes. |
| Exceptions | Use documented API error contract. |
| Security | Backend final authority. |

## Frontend coding standard

| Area | Rule |
|---|---|
| Framework | React with TypeScript. |
| Styling | Tailwind CSS. |
| Server state | TanStack Query. |
| Local workflow state | Zustand. |
| Routing | Guards for auth/role/till session where applicable. |
| Components | Keep reusable UI components separate from business workflow. |
| POS UI | Touchscreen-first, fast, operational. |
| API errors | Use standard error handling. |
| Accessibility | Clear labels, disabled states, visible feedback. |

## Naming standard

| Type | Rule |
|---|---|
| Markdown file | kebab-case. |
| Feature folder | kebab-case business name. |
| Backend class | PascalCase. |
| Backend interface | `IName`. |
| Request/response | Name by use case. |
| Frontend component | PascalCase. |
| Frontend hook | `useXxx`. |
| Zustand store | `*.store.ts`. |
| API client | Feature-scoped API file or core API utility. |

## Quality gates

Before code is accepted:

- [ ] It compiles.
- [ ] It respects architecture boundaries.
- [ ] It does not introduce CQRS/Mediator.
- [ ] It has validation and error handling.
- [ ] It preserves tenant isolation.
- [ ] It handles permission/feature access.
- [ ] It avoids unrelated changes.
- [ ] It updates tests and docs when behavior changes.

## Prohibited code patterns

- Hard-coded tenant IDs or outlet IDs.
- Business logic in controllers.
- Business logic in repositories.
- Direct SQL without documented reason.
- Frontend-only authorization.
- Duplicate final tax/price/payment logic in frontend.
- Silent catch-and-ignore for payment, sync, stock, or audit failures.
- New folder architecture not defined in the 2nd Brain.

## Gate checklist

- [ ] Backend rules read if backend code changes.
- [ ] Frontend rules read if frontend code changes.
- [ ] API rules read if contract changes.
- [ ] Data rules read if persistence changes.
- [ ] Security rules read if access/payment/offline/customer data changes.
- [ ] Tests considered.
- [ ] Feature history considered.
