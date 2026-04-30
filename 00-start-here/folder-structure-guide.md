---
title: Folder Structure Guide
owner: Documentation Lead
status: production-ready
last_reviewed: 2026-04-30
tags:
  - folder-structure
  - navigation
  - obsidian
  - github
---

# Folder Structure Guide

This file explains how the Unified Commerce 2nd Brain is organized and where each type of content belongs.

The structure is designed for Obsidian, GitHub, developers, architects, QA engineers, product owners, support teams, and AI IDE tools.

---

## Root structure

```text
unified-commerce-2nd-brain/
├── 00-start-here/
├── 01-product/
├── 02-architecture/
├── 03-data/
├── 04-api/
├── 05-backend/
├── 06-frontend/
├── 07-modules/
├── 08-user-flows/
├── 09-security-and-compliance/
├── 10-testing-quality/
├── 11-delivery-and-operations/
├── 12-templates/
├── 13-project-history/
├── 14-ai-ide-rules/
├── 99-archive/
├── prompt/
└── README.md
```

---

## Reading order

| Order | Folder | Why |
|---:|---|---|
| 1 | [[00-start-here]] | Entry point and rules. |
| 2 | [[12-templates]] | Required writing formats. |
| 3 | [[01-product]] | Production scope and actors. |
| 4 | [[02-architecture]] | System design and boundaries. |
| 5 | [[03-data]] | Database and entity rules. |
| 6 | [[09-security-and-compliance]] | Tenant isolation, RBAC, payment, OTP, audit. |
| 7 | [[04-api]] | API standards and endpoint rules. |
| 8 | [[05-backend]] | .NET implementation rules. |
| 9 | [[06-frontend]] | React implementation rules. |
| 10 | [[07-modules]] | Feature ownership and specs. |
| 11 | [[08-user-flows]] | Actor-based workflows. |
| 12 | [[10-testing-quality]] | Test strategy and test cases. |
| 13 | [[11-delivery-and-operations]] | Production runbooks. |
| 14 | [[14-ai-ide-rules]] | AI IDE implementation gates. |
| 15 | [[13-project-history]] | Changelog, release notes, bugs, link reports. |
| 16 | [[99-archive]] | Old/superseded content. |

---

## Folder ownership map

| Folder | Owns |
|---|---|
| [[00-start-here]] | Entry, documentation rules, structure, source alignment, readiness, inventory. |
| [[01-product]] | Product vision, business objectives, production scope, modules, roles and actors. |
| [[02-architecture]] | System overview, tenancy, RBAC, backend, frontend, offline, integration, security, scalability. |
| [[03-data]] | Database overview, entity references, tenant consistency, indexing, migration, seed data, EF Core notes. |
| [[04-api]] | API design, request/response, errors, tenant context, feature access, idempotency, concurrency. |
| [[05-backend]] | Clean Architecture, services, validators, DTOs, mapping, repositories, transactions, domain services. |
| [[06-frontend]] | React structure, routing, guards, TanStack Query, Zustand, POS UI, offline, peripherals. |
| [[07-modules]] | Module README files, feature specs, feature histories. |
| [[08-user-flows]] | Step-by-step actor workflows. |
| [[09-security-and-compliance]] | Authentication, authorization, OTP, tenant isolation, device/payment/offline security, audit. |
| [[10-testing-quality]] | Unit, integration, workflow, POS UX, payment, offline, fulfillment, RBAC, reporting tests. |
| [[11-delivery-and-operations]] | Deployment, go-live, tenant onboarding, device provisioning, payment setup, support, incidents. |
| [[12-templates]] | Reusable templates for consistent documentation. |
| [[13-project-history]] | Change log, bug log, release notes, production alignment log. |
| [[14-ai-ide-rules]] | AI IDE reading order, backend/frontend/full-stack gates, bug-fix workflow. |
| [[99-archive]] | Superseded or historical material. |

---

## 00-start-here files

| File | Purpose |
|---|---|
| [[00-start-here/README]] | First file to read. |
| [[00-start-here/documentation-rules]] | Writing and source-alignment rules. |
| [[00-start-here/folder-structure-guide]] | This structure guide. |
| [[00-start-here/source-document-alignment]] | Source-to-folder mapping. |
| [[00-start-here/production-readiness-index]] | Readiness gates. |
| [[00-start-here/file-inventory]] | Navigation inventory. |

Do not place feature implementation details here.

---

## 07-modules structure

Each production module should follow this structure:

```text
07-modules/[module-name]/
├── README.md
└── features/
    └── [feature-name]/
        ├── feature-spec.md
        └── feature-history.md
```

Module README files explain ownership.

Feature specs explain implementation behavior.

Feature history files track changes, bugs, fixes, and notes.

---

## Production module folders

| Module | Purpose |
|---|---|
| [[07-modules/tenant-management]] | Tenants, outlets, document sequences. |
| [[07-modules/platform-administration]] | Platform users, tenant onboarding, tenant lifecycle. |
| [[07-modules/identity-access]] | Staff users, roles, permissions, outlet assignment, feature access. |
| [[07-modules/settings-configuration]] | Tenant settings, feature flags, UI themes. |
| [[07-modules/catalog]] | Products, variants, categories, brands, suppliers, attributes, images. |
| [[07-modules/tax]] | Tax classes and tax rates. |
| [[07-modules/pricing]] | Price lists and outlet overrides. |
| [[07-modules/inventory]] | Balances, stock movements, reservations, transfers, stocktakes. |
| [[07-modules/pos-devices-hardware]] | POS devices, tills, scanners, printers, peripherals. |
| [[07-modules/sales-pos]] | POS checkout, till sessions, cash movements, sales. |
| [[07-modules/payments]] | Payments, transactions, allocations, refunds. |
| [[07-modules/discounts-promotions]] | Discount requests, coupons, redemptions, approvals. |
| [[07-modules/customers]] | Customer profiles, addresses, auth. |
| [[07-modules/ecommerce-orders]] | Carts, orders, wishlists, product reviews. |
| [[07-modules/order-workflow]] | Order, payment, fulfillment transitions and history. |
| [[07-modules/fulfillment-logistics]] | Delivery methods, zones, deliveries, tracking. |
| [[07-modules/returns-exchanges]] | Returns, exchanges, refund/payment allocation. |
| [[07-modules/receipts]] | Receipt templates, generation, print logs. |
| [[07-modules/offline-sync]] | Sync batches, items, queues, conflicts, audit logs. |
| [[07-modules/reporting]] | Reporting summaries and dashboards. |
| [[07-modules/loyalty]] | Loyalty programs, tiers, memberships, transactions. |
| [[07-modules/otp-auth-security]] | OTP channels, purposes, verification. |
| [[07-modules/data-import-ai]] | Import jobs, review, duplicate detection, AI extraction. |
| [[07-modules/audit-compliance]] | Audit coverage and sensitive business events. |

---

## Backend folder alignment

Backend docs must reflect this logical layer structure:

```text
src/
├── POS.Api/
├── POS.Application/
├── POS.Domain/
└── POS.Infrastructure/
```

Do not place `POS.Application`, `POS.Domain`, or `POS.Infrastructure` inside `POS.Api` in documentation.

Related:

- [[05-backend/backend-folder-structure]]
- [[05-backend/clean-architecture-rules]]
- [[02-architecture/backend-architecture]]

---

## Frontend folder alignment

Frontend docs must reflect this structure:

```text
src/
├── bootstrap/
├── core/
├── features/
├── shells/
├── pages/
├── state/
└── shared-kernel/
```

Related:

- [[06-frontend/frontend-folder-structure]]
- [[06-frontend/state-management-rules]]
- [[06-frontend/pos-ui-rules]]

---

## User-flow folders

| Folder | Actor group |
|---|---|
| [[08-user-flows/platform-admin]] | Platform admin workflows. |
| [[08-user-flows/tenant-admin]] | Tenant admin workflows. |
| [[08-user-flows/cashier]] | POS cashier workflows. |
| [[08-user-flows/manager]] | Approval and exception workflows. |
| [[08-user-flows/inventory-staff]] | Stock operation workflows. |
| [[08-user-flows/ecommerce-customer]] | Online customer workflows. |
| [[08-user-flows/ecommerce-ops]] | E-commerce operation workflows. |

User flows should link to module specs and test cases.

---

## Incorrect placement examples

| Incorrect | Correct |
|---|---|
| POS checkout feature spec in [[06-frontend]] | [[07-modules/sales-pos]] |
| Stock movement rules only in API docs | [[03-data/entities/inventory-entities]] and [[07-modules/inventory]] |
| Printer setup only in POS checkout | [[07-modules/pos-devices-hardware]] |
| Refund rules only in returns module | [[07-modules/payments]] with links from returns/exchanges. |
| Offline conflict rules only in frontend | [[02-architecture/offline-first-architecture]] and [[07-modules/offline-sync]] |

---

## Placement decision guide

```text
Scope → 01-product
Architecture → 02-architecture
Data/entity → 03-data
API → 04-api
Backend → 05-backend
Frontend → 06-frontend
Feature → 07-modules
Workflow → 08-user-flows
Security → 09-security-and-compliance
Testing → 10-testing-quality
Operations → 11-delivery-and-operations
Template → 12-templates
History → 13-project-history
AI IDE → 14-ai-ide-rules
Old content → 99-archive
```

---

## Reorganization checklist

Before moving or adding files:

- [ ] Folder ownership is clear.
- [ ] File name is lowercase kebab-case.
- [ ] Duplicate ownership is avoided.
- [ ] Related wiki links are added.
- [ ] [[00-start-here/file-inventory]] is updated if needed.
- [ ] Meaning-changing structure changes are recorded in [[13-project-history/production-alignment-change-log]].
