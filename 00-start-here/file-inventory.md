---
title: File Inventory
owner: Documentation Lead
status: production-ready
last_reviewed: 2026-04-30
tags:
  - file-inventory
  - navigation
  - documentation-map
---

# File Inventory

This file is a navigation index for the Unified Commerce 2nd Brain.

It helps humans and AI IDE tools find the right file before writing documentation or code.

---

## Entry files

| File | Purpose |
|---|---|
| [[README]] | Root overview. |
| [[prompt/prompt]] | Prompt guidance for AI tools. |
| [[00-start-here/README]] | First file to read. |
| [[00-start-here/documentation-rules]] | Writing and alignment rules. |
| [[00-start-here/folder-structure-guide]] | Folder ownership and order. |
| [[00-start-here/source-document-alignment]] | Source document mapping. |
| [[00-start-here/production-readiness-index]] | Readiness gates. |
| [[00-start-here/file-inventory]] | This navigation file. |

---

## Foundation files

| Folder | Key files |
|---|---|
| [[01-product]] | [[01-product/project-scope]], [[01-product/production-module-catalog]], [[01-product/user-roles-and-actors]], [[01-product/business-objectives]], [[01-product/product-vision]] |
| [[02-architecture]] | [[02-architecture/system-overview]], [[02-architecture/tenancy-architecture]], [[02-architecture/role-permission-capability-model]], [[02-architecture/backend-architecture]], [[02-architecture/frontend-architecture]], [[02-architecture/offline-first-architecture]] |
| [[03-data]] | [[03-data/database-overview]], [[03-data/entity-relationship-map]], [[03-data/tenant-consistency-rules]], [[03-data/offline-sync-data-model]], [[03-data/required-schema-extensions]], [[03-data/ef-core-implementation-notes]] |
| [[09-security-and-compliance]] | [[09-security-and-compliance/authorization-model]], [[09-security-and-compliance/data-isolation-controls]], [[09-security-and-compliance/payment-security-rules]], [[09-security-and-compliance/offline-data-protection]], [[09-security-and-compliance/audit-requirements]] |

---

## Entity reference files

| Entity file | Purpose |
|---|---|
| [[03-data/entities/platform-admin-entities]] | Platform admin entities. |
| [[03-data/entities/tenant-outlet-entities]] | Tenants, outlets, addresses, document sequences. |
| [[03-data/entities/identity-access-entities]] | Users, roles, permissions, feature access. |
| [[03-data/entities/product-catalog-entities]] | Catalog, products, variants, attributes, suppliers. |
| [[03-data/entities/inventory-entities]] | Stock balances, movements, reservations, transfers, stocktakes. |
| [[03-data/entities/pos-sales-entities]] | POS devices, tills, sessions, cash movements, sales. |
| [[03-data/entities/customer-entities]] | Customers, auth, OTP, addresses, memberships. |
| [[03-data/entities/ecommerce-entities]] | Carts, orders, wishlists, reviews. |
| [[03-data/entities/logistics-entities]] | Delivery, pickup, zones, tracking. |
| [[03-data/entities/payments-entities]] | Payments, transactions, allocations, refunds. |
| [[03-data/entities/discounts-coupons-entities]] | Discounts, coupons, approvals. |
| [[03-data/entities/returns-exchanges-entities]] | Returns, exchanges, allocations. |
| [[03-data/entities/tax-receipt-audit-configuration-entities]] | Tax, receipts, audit, configuration. |
| [[03-data/entities/reporting-entities]] | Reporting read models. |
| [[03-data/entities/offline-sync-entities]] | Offline sync tables. |
| [[03-data/entities/data-import-ai-entities]] | Import/AI entities or gaps. |
| [[03-data/entities/pos-device-hardware-entities]] | POS hardware entities or gaps. |

---

## API files

| API area | File |
|---|---|
| Overview | [[04-api/api-overview]] |
| Endpoint design | [[04-api/endpoint-design]] |
| Module map | [[04-api/module-endpoint-map]] |
| Request/response | [[04-api/request-response-standard]] |
| Errors | [[04-api/error-contract]] |
| Auth | [[04-api/auth-and-authorization]] |
| Tenant context | [[04-api/tenant-context-api-rules]] |
| Feature access | [[04-api/feature-access-api-rules]] |
| Idempotency | [[04-api/idempotency-rules]] |
| Concurrency | [[04-api/concurrency-rules]] |
| Offline sync | [[04-api/offline-sync-api-rules]] |
| Payment/refund | [[04-api/payment-refund-api-rules]] |
| Order workflow | [[04-api/order-workflow-api-rules]] |
| Device/session | [[04-api/device-session-api-rules]] |

---

## Backend files

| Backend area | File |
|---|---|
| Overview | [[05-backend/backend-overview]] |
| Structure | [[05-backend/backend-folder-structure]] |
| Clean Architecture | [[05-backend/clean-architecture-rules]] |
| Auth | [[05-backend/authentication-authorization]] |
| Feature access | [[05-backend/feature-access-handling]] |
| Validation | [[05-backend/validation-rules]] |
| DTO and mapping | [[05-backend/dto-handling]], [[05-backend/mapping-rules]] |
| Domain services | [[05-backend/domain-service-rules]] |
| Transactions | [[05-backend/transaction-boundary-rules]] |
| Offline sync | [[05-backend/offline-sync-backend-rules]] |
| Payments | [[05-backend/payment-gateway-integration-rules]] |
| Jobs/events | [[05-backend/background-job-rules]], [[05-backend/outbox-event-rules]] |
| Checklist | [[05-backend/backend-implementation-checklist]] |

---

## Frontend files

| Frontend area | File |
|---|---|
| Overview | [[06-frontend/frontend-overview]] |
| Structure | [[06-frontend/frontend-folder-structure]] |
| React rules | [[06-frontend/react-architecture-rules]] |
| Routing/guards | [[06-frontend/routing-and-guards]] |
| API/query | [[06-frontend/api-client-and-query-rules]] |
| State | [[06-frontend/state-management-rules]] |
| POS terminal | [[06-frontend/pos-terminal-state-rules]] |
| POS UI | [[06-frontend/pos-ui-rules]], [[06-frontend/epos-ui-ux-implementation-guide]] |
| Offline | [[06-frontend/offline-frontend-rules]] |
| Peripherals | [[06-frontend/scanner-printer-integration]] |
| E-Commerce | [[06-frontend/ecommerce-storefront-rules]] |
| Fulfillment | [[06-frontend/fulfillment-ops-ui-rules]] |
| Reporting | [[06-frontend/reporting-dashboard-rules]] |
| Feature access | [[06-frontend/feature-access-ui-rules]] |
| Theme/config | [[06-frontend/theme-and-configuration-rules]] |
| UI gate | [[06-frontend/ui-ux-page-design-rules]] |
| Checklist | [[06-frontend/frontend-implementation-checklist]] |

---

## Module root files

| Module | README |
|---|---|
| Tenant management | [[07-modules/tenant-management/README]] |
| Platform administration | [[07-modules/platform-administration/README]] |
| Identity access | [[07-modules/identity-access/README]] |
| Settings configuration | [[07-modules/settings-configuration/README]] |
| Catalog | [[07-modules/catalog/README]] |
| Tax | [[07-modules/tax/README]] |
| Pricing | [[07-modules/pricing/README]] |
| Inventory | [[07-modules/inventory/README]] |
| POS devices/hardware | [[07-modules/pos-devices-hardware/README]] |
| Sales POS | [[07-modules/sales-pos/README]] |
| Payments | [[07-modules/payments/README]] |
| Discounts/promotions | [[07-modules/discounts-promotions/README]] |
| Customers | [[07-modules/customers/README]] |
| E-commerce orders | [[07-modules/ecommerce-orders/README]] |
| Order workflow | [[07-modules/order-workflow/README]] |
| Fulfillment logistics | [[07-modules/fulfillment-logistics/README]] |
| Returns/exchanges | [[07-modules/returns-exchanges/README]] |
| Receipts | [[07-modules/receipts/README]] |
| Offline sync | [[07-modules/offline-sync/README]] |
| Reporting | [[07-modules/reporting/README]] |
| Loyalty | [[07-modules/loyalty/README]] |
| OTP auth security | [[07-modules/otp-auth-security/README]] |
| Data import AI | [[07-modules/data-import-ai/README]] |
| Audit compliance | [[07-modules/audit-compliance/README]] |

---

## User flow folders

| Folder | Actor group |
|---|---|
| [[08-user-flows/platform-admin]] | Platform admin. |
| [[08-user-flows/tenant-admin]] | Tenant admin. |
| [[08-user-flows/cashier]] | Cashier. |
| [[08-user-flows/manager]] | Manager/approver. |
| [[08-user-flows/inventory-staff]] | Inventory staff. |
| [[08-user-flows/ecommerce-customer]] | Online customer. |
| [[08-user-flows/ecommerce-ops]] | E-commerce operations. |

---

## Testing and operations files

| Area | Important files |
|---|---|
| Testing | [[10-testing-quality/test-strategy]], [[10-testing-quality/workflow-test-cases]], [[10-testing-quality/offline-sync-test-cases]], [[10-testing-quality/payment-refund-test-cases]], [[10-testing-quality/release-readiness-checklist]] |
| Operations | [[11-delivery-and-operations/deployment-checklist]], [[11-delivery-and-operations/production-go-live-checklist]], [[11-delivery-and-operations/tenant-onboarding-runbook]], [[11-delivery-and-operations/pos-device-provisioning-runbook]], [[11-delivery-and-operations/offline-sync-support-runbook]], [[11-delivery-and-operations/incident-response]] |
| Templates | [[12-templates/feature-spec-template]], [[12-templates/module-readme-template]], [[12-templates/user-flow-template]], [[12-templates/api-spec-template]], [[12-templates/entity-reference-template]], [[12-templates/test-case-template]] |
| History | [[13-project-history/overall-changelog]], [[13-project-history/production-alignment-change-log]], [[13-project-history/overall-bugs]], [[13-project-history/release-notes]], [[13-project-history/link-check-report]] |
| AI IDE | [[14-ai-ide-rules/ai-ide-project-understanding]], [[14-ai-ide-rules/backend-implement]], [[14-ai-ide-rules/frontend-implement]], [[14-ai-ide-rules/fullstack-feature-implementation-rule]], [[14-ai-ide-rules/database-alignment-rule]] |

---

## Quick lookup

| Question | Start with |
|---|---|
| What are we building? | [[01-product/project-scope]] |
| What tables exist? | [[03-data/database-overview]] |
| How does RBAC work? | [[02-architecture/role-permission-capability-model]] |
| How does tenant isolation work? | [[03-data/tenant-consistency-rules]] |
| How should backend code be structured? | [[05-backend/backend-folder-structure]] |
| How should frontend code be structured? | [[06-frontend/frontend-folder-structure]] |
| How does offline POS work? | [[02-architecture/offline-first-architecture]] |
| How are payments/refunds handled? | [[04-api/payment-refund-api-rules]] |
| What should AI IDE read first? | [[14-ai-ide-rules/ai-ide-project-understanding]] |

---

## Maintenance checklist

Update this file when:

- [ ] A root folder is added.
- [ ] A production module is added.
- [ ] A major rule file is added.
- [ ] A file is renamed or moved.
- [ ] AI IDE reading order changes.
- [ ] Source document alignment changes.
