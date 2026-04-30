---
title: Frontend Implementation Rule
owner: Frontend Lead
status: production-ready
last_reviewed: 2026-04-30
tags: [ai-ide, frontend, implementation]
---

# Frontend Implementation Rule

## Purpose

This file tells AI IDE tools how to implement frontend work for the Unified Commerce system.

Use React, TypeScript, Tailwind CSS, TanStack Query, Zustand, and the uploaded frontend architecture. Keep POS screens touchscreen-first and operational.


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


## Frontend source architecture

The uploaded frontend structure defines:

- `bootstrap/` for main app, router, guards, providers, layouts.
- `core/` for API, auth, offline, peripherals, config, utilities.
- `features/` for feature modules.
- `shells/` for POS screen composition.
- `pages/` for routed pages.
- `state/` for Zustand stores and cart orchestrator.
- `shared-kernel/` for money, tax, pricing, receipt, and invoice helpers.

## Implementation sequence

1. Read [[14-ai-ide-rules/frontend-implementation-documentation-gate-rule]].
2. Identify affected feature and page/shell/store/API area.
3. Read API docs and response/error contract.
4. Read feature spec and user flow.
5. Identify server state versus local workflow state.
6. Implement typed API client hook with TanStack Query where server data is involved.
7. Implement Zustand state only for local UI/workflow state.
8. Implement UI with Tailwind and existing component rules.
9. Add loading, empty, error, offline, permission, and disabled states.
10. Update tests and documentation.

## State ownership

| State type | Owner |
|---|---|
| Products, orders, payments from server | TanStack Query. |
| Current POS cart | Zustand/cart store or orchestrator. |
| Session/till/device state | Session/till stores and guards. |
| Offline queue status | Offline store and core offline utilities. |
| UI panel/modal state | Zustand/UI state or local component state. |
| Feature permissions | Auth/session context plus backend authority. |

## POS screen rules

POS screens must prioritize:

- Always-focused scan/search field.
- Fast product add.
- Visible basket and totals.
- Large tap targets.
- Clear payment action.
- Hold/recall/discount/return actions where documented.
- Offline and printer/device/session status visibility.
- Minimal decorative website-style layout.

## Frontend validation rule

Frontend validation helps the user avoid mistakes, but backend is final authority.

Frontend may validate:

- Required fields.
- Input format.
- Negative or zero values.
- Disabled actions without session.
- Missing selected item/customer/payment method.

Frontend must not be final authority for:

- Permission checks.
- Feature access.
- Stock availability.
- Payment capture.
- Refund eligibility.
- Tax/discount final totals.
- Offline sync acceptance.

## API handling rule

Use documented API conventions:

- `/api/v1` versioning.
- Standard request/response patterns.
- Error contract.
- Pagination/filtering/sorting where applicable.
- Idempotency for duplicate-risk operations.
- Tenant/outlet/device context where required.

## Peripheral rule

Use `core/peripherals` abstractions for printer, scanner, and cash drawer behavior.

Do not mix raw printer/scanner behavior into POS page components unless the frontend architecture explicitly supports it.

## Not allowed

- Do not hard-code API URLs outside API client configuration.
- Do not duplicate backend business rules as final truth.
- Do not store server source of truth only in Zustand.
- Do not create UI flows not present in user-flow/module docs.
- Do not invent screens or workflows.
- Do not hide security logic only in frontend.

## Completion checklist

- [ ] Relevant frontend docs read.
- [ ] API contract read.
- [ ] Feature spec and user flow read.
- [ ] Query/state boundaries correct.
- [ ] Touchscreen POS rules followed where applicable.
- [ ] Permission/feature UI states included.
- [ ] Offline/peripheral states included where relevant.
- [ ] Backend remains final authority.
- [ ] Documentation and tests updated.
