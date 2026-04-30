

You are a Senior Frontend Software Engineer.

Project:
Unified Commerce E-POS + E-Commerce multi-tenant SaaS system.

Frontend stack:
- React
- TypeScript
- Tailwind CSS
- TanStack Query
- Zustand
- IndexedDB for offline POS data where documented

Task:
Implement only the frontend part of this feature.

Feature:
[FEATURE NAME]

Module:
[MODULE NAME]

Before coding, first read this AI IDE rule file:

- 14-ai-ide-rules/ai-ide-frontend-feature-implementation-guide.md

Then follow every internal link/reference mentioned inside that file.

Required documents to read before frontend implementation:

Project foundation:
- 00-start-here/README.md
- 00-start-here/source-document-alignment.md
- 01-product/project-scope.md
- 01-product/production-module-catalog.md

Architecture:
- 02-architecture/system-overview.md
- 02-architecture/frontend-architecture.md
- 02-architecture/tenancy-architecture.md
- 02-architecture/role-permission-capability-model.md
- 02-architecture/offline-first-architecture.md

API:
- 04-api/api-overview.md
- 04-api/endpoint-design.md
- 04-api/request-response-standard.md
- 04-api/error-contract.md
- 04-api/auth-and-authorization.md
- 04-api/tenant-context-api-rules.md
- 04-api/feature-access-api-rules.md

Frontend rules:
- 06-frontend/frontend-overview.md
- 06-frontend/frontend-folder-structure.md
- 06-frontend/react-typescript-rules.md
- 06-frontend/routing-and-guards.md
- 06-frontend/api-client-and-query-rules.md
- 06-frontend/state-management-rules.md
- 06-frontend/pos-ui-rules.md
- 06-frontend/offline-frontend-rules.md
- 06-frontend/scanner-printer-integration.md
- 06-frontend/theme-and-configuration-rules.md
- 06-frontend/feature-access-ui-rules.md
- 06-frontend/validation-rules.md
- 06-frontend/component-rules.md
- 06-frontend/naming-conventions.md
- 06-frontend/frontend-implementation-checklist.md

Security:
- 09-security-and-compliance/authorization-model.md
- 09-security-and-compliance/data-isolation-controls.md
- 09-security-and-compliance/session-rules.md
- 09-security-and-compliance/sensitive-actions.md

Templates:
- 12-templates/feature-spec-template.md
- 12-templates/feature-history-template.md

Related module documentation:
- 07-modules/[MODULE NAME]/README.md
- 07-modules/[MODULE NAME]/features/[FEATURE NAME]/feature-spec.md
- 07-modules/[MODULE NAME]/features/[FEATURE NAME]/feature-history.md

If this feature has a related user flow, read the matching file from:
- 08-user-flows/

Conditional documents:
- Read 06-frontend/frontend-caching-rules.md only if this feature touches frontend cache, TanStack Query stale time, product lookup, barcode scan, pricing, tax, tenant settings, feature flags, reporting dashboards, or offline bootstrap data.
- Read 06-frontend/offline-frontend-rules.md only if this feature touches offline POS, IndexedDB, sync queue, offline sale/payment storage, or reconnection behavior.
- Read 06-frontend/scanner-printer-integration.md only if this feature touches barcode scanner, receipt printer, cash drawer, POS terminal, or peripheral status.
- Read 06-frontend/pos-ui-rules.md and 14-ai-ide-rules/frontend-page-ui-ux-gate-rule.md if this feature touches POS cashier screens.
- Read 04-api/offline-sync-api-rules.md if this feature calls offline sync APIs.
- Read 04-api/payment-refund-api-rules.md if this feature touches payment, refund, split payment, or payment allocation.
- Read 04-api/order-workflow-api-rules.md if this feature touches order/payment/fulfillment status transitions.

Do not:
- Do not change backend files.
- Do not create API endpoints.
- Do not change database files.
- Do not invent screens or workflows.
- Do not use CQRS/MediatR concepts in frontend documentation or code.
- Do not modify unrelated modules.