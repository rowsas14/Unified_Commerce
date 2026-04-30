---
title: POS Devices and Hardware Module
owner: Product + Engineering
status: production-ready-module-doc
last_reviewed: 2026-04-30
tags:
  - unified-commerce
  - module
  - pos-devices-hardware
---

# POS Devices and Hardware Module

## Purpose

Controls POS terminal identity, till assignment, outlet context, and documented peripheral behavior for scanners, printers, and cash drawer integration.

This module is part of the production-ready Unified Commerce E-POS + E-Commerce SaaS system. It must follow tenant isolation, role/permission validation, feature access validation, API rules, backend Clean Architecture rules, frontend implementation rules, audit requirements, and testing expectations defined in the 2nd Brain.

## Required reading

| Area | Document |
|---|---|
| Product scope | [[01-product/project-scope|Project Scope]] |
| Module catalog | [[01-product/production-module-catalog|Production Module Catalog]] |
| System architecture | [[02-architecture/system-overview|System Overview]] |
| Tenancy model | [[02-architecture/tenancy-model|Tenancy Model]] |
| RBAC / feature access | [[02-architecture/role-permission-capability-model|Role Permission Capability Model]] |
| Database overview | [[03-data/database-overview|Database Overview]] |
| API rules | [[04-api/api-overview|API Overview]] |
| Backend rules | [[05-backend/backend-overview|Backend Overview]] |
| Frontend rules | [[06-frontend/frontend-overview|Frontend Overview]] |
| Security rules | [[09-security-and-compliance/README|Security and Compliance]] |
| AI IDE rules | [[14-ai-ide-rules/README|AI IDE Rules]] |

## Actors

- Tenant Admin
- Outlet Manager
- Cashier
- Frontend App
- Backend Service

## Module scope

### In scope

- Maintain only the responsibilities explicitly described in this module and its feature specs.
- Use only database tables listed in this module or referenced by linked modules.
- Enforce tenant isolation for every tenant-owned operation.
- Apply backend authorization and feature access checks before changing data.
- Update audit logs or workflow history where the uploaded database design provides the required table.
- Keep frontend screens operational and workflow-focused, especially for POS-facing behavior.

### Out of scope

- Do not own responsibilities of unrelated modules.
- Do not create new tables that are not in the uploaded database design.
- Do not add CQRS, mediator, event sourcing, or undocumented architecture patterns.
- Do not bypass the API/backend/security rules documented in the 2nd Brain.


## Current schema gaps

- No approved separate printer, scanner, cash drawer device, printer profile, or test print table exists in the uploaded database design. Document peripheral behavior only through frontend/peripheral rules unless schema is later approved.

## Owned or primary tables

| Table | PK | Important FK / attribute references | Purpose |
| --- | --- | --- | --- |
| `tills` | `id` | tenant_id -> tenants.id; outlet_id -> outlets.id | Cash register/till master. |
| `pos_devices` | `id` | tenant_id -> tenants.id; outlet_id -> outlets.id; till_id -> tills.id | Registered POS terminal/browser/device. |

## Feature map

| Feature | Purpose | Primary tables |
| --- | --- | --- |
| [[07-modules/pos-devices-hardware/features/tills/feature-spec|Tills]] | Cash register/till master assigned to tenant and outlet. | `tills`, `outlets` |
| [[07-modules/pos-devices-hardware/features/pos-device-registration/feature-spec|POS Device Registration]] | Registered POS terminal/browser/device for online/offline POS. | `pos_devices`, `tills`, `outlets` |
| [[07-modules/pos-devices-hardware/features/terminal-outlet-assignment/feature-spec|Terminal Outlet Assignment]] | Ensures device outlet context matches assigned till outlet before POS operation. | `pos_devices`, `tills`, `outlets` |
| [[07-modules/pos-devices-hardware/features/scanner-printer-configuration/feature-spec|Scanner and Printer Configuration]] | Documents configured scanner/printer behavior from scope and frontend architecture without inventing unapproved tables. | `pos_devices` |

## Dependency map

```text
Tenant context
  -> permissions and feature access
  -> module validation
  -> API request handling
  -> service layer workflow
  -> repository/data access
  -> audit/reporting where applicable
```

## Business rule summary

- Tenant-owned rows must never be shared across tenants.
- Status changes must follow documented status rules and allowed transitions where available.
- Monetary, stock, payment, refund, discount, receipt, offline sync, and audit-related changes must be validated by the backend.
- Frontend may guide the user and protect the workflow, but backend remains the final authority.
- Read-model/reporting tables are not source-of-truth transaction tables.
- Offline staging tables are not source-of-truth transaction tables.

## API reference policy

This module must follow [[04-api/endpoint-design|Endpoint Design]] and [[04-api/api-overview|API Overview]]. Do not invent endpoint names in implementation. Final API routes must be documented in the API folder before code is written.

## Backend implementation policy

- Use Clean Architecture with Service Pattern and Repository Pattern only.
- Controllers must remain thin.
- Application services orchestrate workflows.
- Domain entities/domain services hold pure business rules where applicable.
- Repositories handle data access.
- Unit of Work/transaction boundaries must protect multi-table writes.

## Frontend implementation policy

- Use React + TypeScript + Tailwind CSS.
- Use TanStack Query for server state.
- Use Zustand only for client workflow state.
- Never trust UI hiding as authorization.
- Keep POS-facing screens touchscreen-first, fast, and operational.

## QA checklist

- [ ] Tenant isolation tested.
- [ ] Permission and feature access tested.
- [ ] Required entities and FK relationships verified.
- [ ] API request/response validation tested.
- [ ] Backend service workflow tested.
- [ ] Frontend loading/error/empty states tested.
- [ ] Audit or history behavior tested where applicable.
- [ ] Offline behavior tested if the feature can operate offline.
- [ ] Reporting impact reviewed where applicable.

## Implementation notes for AI IDE tools

- Read this module README before editing any feature in this module.
- Read the target feature `feature-spec.md` and `feature-history.md` before coding.
- Do not implement schema gaps as new tables unless the database design is officially updated.
- Update feature history after implementation, bug fixes, or rule changes.
- Cross-reference related modules instead of duplicating rules.
