---
title: Frontend Implementation Checklist
folder: 06-frontend
status: production-ready
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_frontend_architecture: Frontend archi V1.txt
source_backend_architecture: Back end Architecture final.txt
last_reviewed: 2026-04-30
tags: [frontend, react, typescript, tailwind, pos, ecommerce, unified-commerce]
---

# Frontend Implementation Checklist

Use this checklist before implementing or changing any frontend feature in the Unified Commerce system.
It is designed for frontend developers and AI IDE tools.

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

## 1. Documentation readiness

- [ ] Read [[01-product/project-scope]].
- [ ] Read relevant module in [[07-modules/README]].
- [ ] Read related user flow in [[08-user-flows/README]].
- [ ] Read [[02-architecture/frontend-architecture]].
- [ ] Read [[04-api/api-overview]] and relevant API rule file.
- [ ] Read [[05-backend/backend-overview]] for backend authority.
- [ ] Read relevant security rule in [[09-security-and-compliance/README]].
- [ ] Confirm feature exists in uploaded scope/current 2nd Brain.

## 2. Scope boundary

- [ ] Feature belongs to a documented production module.
- [ ] No new entity/table is assumed from frontend.
- [ ] No new API endpoint is invented without API documentation.
- [ ] No new workflow is created without user-flow/module documentation.
- [ ] Feature does not reduce the system to MVP/basic behavior.

## 3. Folder placement

- [ ] Page file belongs in `pages/` only if route-level.
- [ ] Feature code belongs in `features/<module>/`.
- [ ] API hooks belong in `features/<module>/api/`.
- [ ] Reusable POS layout belongs in `shells/`.
- [ ] Shared infrastructure belongs in `core/`.
- [ ] Workflow state belongs in `state/` only when justified.

## 4. API integration

- [ ] API path uses `/api/v1`.
- [ ] API client uses `core/api/http.ts`.
- [ ] Query/mutation uses TanStack Query.
- [ ] Tenant/outlet/session/device context is handled.
- [ ] Idempotency key is used where required.
- [ ] Error contract is mapped to UI states.
- [ ] Cache invalidation is explicit.

## 5. State ownership

- [ ] Server data stays in TanStack Query.
- [ ] Active workflow state uses Zustand only when needed.
- [ ] Form-only state stays local/form-managed.
- [ ] Offline payloads use approved offline queue/IndexedDB.
- [ ] State reset is handled on logout/tenant/outlet/session changes.

## 6. Access and security

- [ ] Route guard is correct.
- [ ] Feature access is reflected in UI.
- [ ] Permission-denied state is handled.
- [ ] Feature-disabled state is handled.
- [ ] Frontend does not replace backend authorization.
- [ ] Sensitive data is not stored in browser storage.

## 7. POS-specific checklist

- [ ] UI supports scan → add → pay → print.
- [ ] Scan/search field is prominent.
- [ ] Cart and payable total are always visible.
- [ ] Touch targets are cashier-friendly.
- [ ] Till/session locked state is handled.
- [ ] Payment action is dominant and safe.
- [ ] Printer/scanner failure state is clear.
- [ ] Offline state is visible.

## 8. Offline checklist

- [ ] Offline behavior is documented for the feature.
- [ ] Offline-safe actions are limited.
- [ ] Local transaction IDs are generated where needed.
- [ ] Pending sync state is visible.
- [ ] Sync conflict state is visible.
- [ ] Backend sync acceptance is required before final synced state.

## 9. Validation checklist

- [ ] Required fields are validated in UI.
- [ ] Backend validation errors are displayed.
- [ ] Duplicate submit is prevented.
- [ ] Status transition failure is handled.
- [ ] Payment/refund/stock errors are not hidden.

## 10. UX checklist

- [ ] Loading state exists.
- [ ] Empty state exists.
- [ ] Error state exists.
- [ ] Permission/feature state exists.
- [ ] Offline state exists where applicable.
- [ ] User can recover from failure where business flow allows.
- [ ] UI text is business-readable.

## 11. Testing checklist

- [ ] Success path tested.
- [ ] Validation error tested.
- [ ] Permission denied tested.
- [ ] Feature disabled tested.
- [ ] Offline/network failure tested where relevant.
- [ ] Backend rejection tested.
- [ ] POS touch/scan behavior tested where relevant.

## 12. Documentation update checklist

- [ ] Feature spec updated if behavior changed.
- [ ] Feature history updated.
- [ ] User flow updated if workflow changed.
- [ ] API docs updated if endpoint contract changed.
- [ ] Test documentation updated.
- [ ] AI IDE prompt/rules updated if implementation flow changed.

## Final rule

Do not implement frontend code before the feature's scope, API, backend authority, state owner and UI states are clear.
