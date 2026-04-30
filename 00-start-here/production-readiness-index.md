---
title: Production Readiness Index
owner: Documentation Lead
status: production-ready
last_reviewed: 2026-04-30
tags:
  - production-readiness
  - quality-gate
  - implementation-readiness
---

# Production Readiness Index

This file defines how to judge whether a folder, module, feature spec, or implementation area is ready for production work.

A file is not ready just because it exists. It is ready only when it is source-aligned, specific, linked, and usable.

---

## Readiness levels

| Level | Meaning | Implement from it? |
|---|---|---|
| Not Started | Missing or empty. | No |
| Draft | Content exists but is incomplete or generic. | No |
| Source Aligned | Matches source docs but lacks implementation detail. | Review only |
| Implementation Ready | Developers can safely implement from it. | Yes |
| Production Ready | Covers implementation, QA, security, audit, offline, and operations where relevant. | Yes |
| Superseded | Replaced by another file. | No |

---

## Global readiness gates

The 2nd Brain is production-ready only when:

- [ ] Source documents are reflected correctly.
- [ ] Important internal links work.
- [ ] Every production module has a clear folder.
- [ ] Every production feature has a meaningful `feature-spec.md`.
- [ ] Database docs use exact table names.
- [ ] Tenant isolation is documented.
- [ ] RBAC and feature access are consistently documented.
- [ ] Offline behavior is documented for offline-capable workflows.
- [ ] Financial and inventory source-of-truth rules are documented.
- [ ] QA test cases cover workflows and edge cases.
- [ ] Operations runbooks exist for production support.
- [ ] AI IDE rules point to correct source-aligned docs.

---

## Folder readiness matrix

| Folder | Target readiness | Primary dependency |
|---|---|---|
| [[00-start-here]] | Production Ready | Source docs and structure. |
| [[12-templates]] | Production Ready | Documentation rules. |
| [[01-product]] | Production Ready | Scope document. |
| [[02-architecture]] | Production Ready | Scope, database, backend, frontend sources. |
| [[03-data]] | Production Ready | Database design. |
| [[09-security-and-compliance]] | Production Ready | Scope, RBAC, OTP, payment, offline, audit. |
| [[04-api]] | Implementation Ready | Data and security docs. |
| [[05-backend]] | Implementation Ready | Backend architecture, API, data, security. |
| [[06-frontend]] | Implementation Ready | Frontend architecture, API, POS UI, offline rules. |
| [[07-modules]] | Implementation Ready | Product, data, API, backend, frontend, security. |
| [[08-user-flows]] | Implementation Ready | Module feature specs. |
| [[10-testing-quality]] | Production Ready | Feature specs and user flows. |
| [[11-delivery-and-operations]] | Production Ready | System design and support needs. |
| [[14-ai-ide-rules]] | Production Ready | Implementation docs. |
| [[13-project-history]] | Production Ready | Change tracking. |
| [[99-archive]] | Controlled | Old/superseded content only. |

---

## Module readiness checklist

Each module under [[07-modules]] must have:

- [ ] `README.md`.
- [ ] Clear purpose and scope.
- [ ] Owned features.
- [ ] Related database tables.
- [ ] Related API docs.
- [ ] Related backend docs.
- [ ] Related frontend docs, if UI exists.
- [ ] Related user flows.
- [ ] Required permissions.
- [ ] Feature access rules.
- [ ] Tenant/outlet/channel context.
- [ ] Audit impact.
- [ ] Offline impact, if relevant.
- [ ] Reporting impact, if relevant.
- [ ] What the module must not own.
- [ ] `feature-spec.md` files.
- [ ] `feature-history.md` files.

---

## Feature spec readiness checklist

Each `feature-spec.md` must include:

- [ ] Purpose.
- [ ] Scope.
- [ ] Actors.
- [ ] Permissions.
- [ ] Feature access rule.
- [ ] Business rules.
- [ ] Validation rules.
- [ ] Status rules, if relevant.
- [ ] Database impact.
- [ ] API impact.
- [ ] Backend impact.
- [ ] Frontend impact.
- [ ] Offline impact, if relevant.
- [ ] Audit impact.
- [ ] Reporting impact.
- [ ] Acceptance criteria.
- [ ] Open questions.

Placeholder feature specs are not ready.

---

## Production module review order

| Order | Module | Reason |
|---:|---|---|
| 1 | [[07-modules/tenant-management]] | All tenant data depends on tenant/outlet context. |
| 2 | [[07-modules/platform-administration]] | Platform enables tenants and features. |
| 3 | [[07-modules/identity-access]] | Roles and permissions gate workflows. |
| 4 | [[07-modules/settings-configuration]] | Settings affect runtime behavior. |
| 5 | [[07-modules/catalog]] | Product and variants are used everywhere. |
| 6 | [[07-modules/tax]] | Checkout, refunds, receipts, reports need tax. |
| 7 | [[07-modules/pricing]] | Sales and carts need pricing. |
| 8 | [[07-modules/inventory]] | POS, orders, returns, fulfillment need stock. |
| 9 | [[07-modules/pos-devices-hardware]] | POS needs device, till, scanner, printer context. |
| 10 | [[07-modules/sales-pos]] | POS depends on catalog, tax, pricing, stock, device, session. |
| 11 | [[07-modules/payments]] | Sales/orders need payments and refunds. |
| 12 | [[07-modules/discounts-promotions]] | Discounts affect totals and tax. |
| 13 | [[07-modules/customers]] | POS and e-commerce need tenant-scoped customers. |
| 14 | [[07-modules/ecommerce-orders]] | Orders depend on catalog, customer, pricing, stock, payment. |
| 15 | [[07-modules/order-workflow]] | Status transitions depend on order/payment/fulfillment. |
| 16 | [[07-modules/fulfillment-logistics]] | Fulfillment depends on orders and stock. |
| 17 | [[07-modules/returns-exchanges]] | Returns depend on sale/order, payment, stock, receipt. |
| 18 | [[07-modules/receipts]] | Receipts depend on transaction data. |
| 19 | [[07-modules/offline-sync]] | Offline sync depends on POS, payments, stock, receipts, device. |
| 20 | [[07-modules/reporting]] | Reporting depends on transactional modules. |
| 21 | [[07-modules/loyalty]] | Loyalty depends on customers and transactions. |
| 22 | [[07-modules/otp-auth-security]] | OTP supports secure login and verification. |
| 23 | [[07-modules/data-import-ai]] | Import affects catalog, customers, suppliers, stock. |
| 24 | [[07-modules/audit-compliance]] | Audit spans sensitive actions across modules. |

---

## Data readiness gates

Data docs are ready when:

- [ ] All schema groups are represented.
- [ ] Exact table names are used.
- [ ] FK ownership is explained.
- [ ] Tenant consistency is documented.
- [ ] Source-of-truth tables are separated from read models.
- [ ] Offline staging tables are not treated as final records.
- [ ] EF Core and PostgreSQL notes exist.
- [ ] Indexing strategy covers tenant, lookup, sync, reporting.
- [ ] Seed data strategy covers reference data.
- [ ] Schema gaps are visible.

---

## API readiness gates

API docs are ready when:

- [ ] Tenant context is defined.
- [ ] Feature access is defined.
- [ ] Backend authorization is required.
- [ ] Request/response standard exists.
- [ ] Error contract covers validation, permission, feature disabled, stock conflict, payment failure, sync conflict, invalid transition.
- [ ] Idempotency covers sale, order, payment, refund, receipt, sync.
- [ ] Concurrency covers stock, coupons, payment/refund, document sequences.
- [ ] Module endpoint map exists.

---

## Backend readiness gates

Backend docs are ready when:

- [ ] Clean Architecture layers are separated.
- [ ] Controllers are thin.
- [ ] Application services orchestrate use cases.
- [ ] Domain services hold pure business logic.
- [ ] Repositories and Unit of Work are documented.
- [ ] Transaction boundaries are defined.
- [ ] Validation is centralized.
- [ ] Feature access is server-side.
- [ ] Offline sync backend rules are defined.
- [ ] Payment/refund integration rules are defined.

---

## Frontend readiness gates

Frontend docs are ready when:

- [ ] React structure matches the uploaded architecture.
- [ ] POS UI supports scan-add-pay speed.
- [ ] Till/session guard behavior is documented.
- [ ] TanStack Query and Zustand responsibilities are separated.
- [ ] Offline queue and connectivity behavior are documented.
- [ ] Scanner/printer/cash drawer boundaries are clear.
- [ ] Frontend hiding is not treated as security.
- [ ] Shared-kernel helpers are not treated as backend authority.

---

## Security, QA, and operations gates

| Area | Must cover |
|---|---|
| Security | Platform/tenant/customer separation, tenant isolation, RBAC, OTP, payment secrets, device security, offline data, audit. |
| QA | Tenant isolation, RBAC, POS, payments, refunds, discounts, returns, inventory, offline sync, orders, fulfillment, receipts, reporting. |
| Operations | Tenant onboarding, device provisioning, payment setup, offline sync support, deployment, go-live, monitoring, backup, incidents. |
| AI IDE | Project reading, backend gate, frontend gate, database rule, production scope rule, offline rule, payment/refund rule. |

---

## Content writing priority

```text
1. 00-start-here
2. 12-templates
3. 01-product
4. 02-architecture
5. 03-data
6. 09-security-and-compliance
7. 04-api
8. 05-backend
9. 06-frontend
10. 07-modules
11. 08-user-flows
12. 10-testing-quality
13. 11-delivery-and-operations
14. 14-ai-ide-rules
15. 13-project-history
16. 99-archive
```

Do not write module features before the foundation folders are ready.
