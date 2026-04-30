---
title: Frontend Coding Standards
folder: 06-frontend
status: production-ready
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_frontend_architecture: Frontend archi V1.txt
source_backend_architecture: Back end Architecture final.txt
last_reviewed: 2026-04-30
tags: [frontend, react, typescript, tailwind, pos, ecommerce, unified-commerce]
---

# Frontend Coding Standards

This document defines coding standards for the React + TypeScript frontend of the Unified Commerce system.
It is intended for human developers and AI IDE tools.

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

## Stack rules

| Area | Required standard |
|---|---|
| Framework | React |
| Language | TypeScript |
| Styling | Tailwind CSS |
| Server state | TanStack Query |
| Workflow state | Zustand |
| Offline POS | IndexedDB through `core/offline` |
| API version | `/api/v1` |

## TypeScript standards

- Prefer explicit types for API and component boundaries.
- Avoid `any` unless technically unavoidable.
- Use `unknown` for untrusted external payloads before parsing.
- Use union types for statuses when status set is known from API/data docs.
- Keep UI models separate from backend entity persistence models.
- Do not expose database table structure directly into components.

## React standards

- Prefer functional components.
- Keep page components thin.
- Keep feature components module-owned.
- Avoid deeply nested prop drilling for workflow state.
- Use hooks for reusable behavior.
- Keep effects explicit and minimal.
- Avoid hidden side effects in render paths.

## Styling standards

Tailwind CSS should be used consistently.

Rules:

- prefer utility classes with readable grouping;
- avoid arbitrary values unless needed;
- use design tokens/theme values where available;
- maintain POS contrast and readability;
- avoid website-style decorative spacing on POS screens;
- use responsive behavior where admin/e-commerce screens require it.

## API standards

- No raw fetch calls in page components.
- Use `core/api/http.ts` and feature API hooks.
- Use TanStack Query for server state.
- Use documented API error contract.
- Include tenant/outlet/session/device context where required.
- Use idempotency keys for high-risk mutations where API rules require.

## State standards

- Use Zustand only for active workflow state.
- Do not store server lists in Zustand by default.
- Reset stores on logout/tenant/outlet/session changes.
- Keep cart/session/offline state clear and testable.

## Error handling standards

Frontend code must handle:

- loading;
- empty;
- validation error;
- permission denied;
- feature disabled;
- conflict;
- offline/network failure;
- backend unexpected failure.

Do not hide backend errors behind generic messages when the API contract provides business detail.

## Security standards

- Do not store secrets in browser storage.
- Do not store plain OTPs or passwords.
- Do not display cross-tenant cached data.
- Do not rely on frontend access checks as security.
- Clear sensitive state on logout/session expiration.

## POS standards

POS frontend code must prioritize:

- fast scan/add flow;
- clear cart state;
- visible total;
- payment readiness;
- touch-friendly controls;
- offline visibility;
- printer/scanner status.

## Comments

Use comments only when they explain non-obvious business/technical constraints.
Good comment targets:

- offline queue edge cases;
- payment idempotency;
- scanner double-input protection;
- tenant/outlet cache reset;
- backend authority boundary;
- printer failure after completed sale.

Avoid comments that repeat obvious code.

## AI IDE rules

AI IDE tools must:

- read relevant 2nd Brain docs before editing;
- not invent frontend modules;
- not create endpoint paths not documented by API rules;
- not bypass backend authority;
- update feature history after changes;
- preserve POS operational UX.

## Checklist

- [ ] Code is typed.
- [ ] API access uses approved client/hook pattern.
- [ ] State owner is correct.
- [ ] UI handles loading/error/empty states.
- [ ] Tenant/outlet/session reset behavior is safe.
- [ ] POS UI remains operational.
- [ ] No unsupported pattern is introduced.
