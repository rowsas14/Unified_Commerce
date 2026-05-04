---
title: Platform User Management Flow
owner: Product + QA
status: draft
last_reviewed: 2026-05-01
tags: [user-flow, platform-admin, platform-users]
source: Unified Commerce production scope + database design
related_docs:
  - [[07-modules/platform-administration/README|Platform Administration Module]]
  - [[07-modules/platform-administration/features/platform-users/feature-spec|Platform Users Feature Spec]]
  - [[02-architecture/tenancy-architecture|Tenancy Architecture]]
  - [[02-architecture/role-permission-capability-model|Role Permission Capability Model]]
  - [[04-api/module-endpoint-map|Module Endpoint Map]]
  - [[04-api/tenant-context-api-rules|Tenant Context API Rules]]
  - [[04-api/error-contract|API Error Contract]]
  - [[09-security-and-compliance/audit-requirements|Audit Requirements]]
---

# Platform User Management Flow

## Actor

Platform Admin / Support User.

## Purpose

Create and manage **platform-side users** (`platform_users`) who operate platform routes (e.g. `/api/v1/platform/...`) for tenant lifecycle support and tenant feature entitlement administration.

## Preconditions

- Actor is authenticated as a **platform user** (not a tenant staff user).
- Actor is authorized to perform platform-user administration.
  - Open Question: exact permission/feature key(s) for this feature are not specified in the current feature spec; do not hard-code until permission catalog / API spec confirms.

## Scope notes

- `platform_users` is **platform-owned** data (no tenant ownership), but actions may target a specific tenant for support workflows.
- Do not treat platform users as tenant staff.
- Do not rely on UI hiding as security; backend enforces access.

## Happy path

### A) View platform users (list/search)

1. Platform Admin navigates to Platform User Management.
2. System requests platform user list.
3. Backend returns platform users with safe fields only.
   - Must not expose password hashes.

Open Question:
- Filtering/sorting/search requirements are not specified yet.

### B) Create a platform user

1. Platform Admin enters new platform user details.
2. System validates required fields and uniqueness constraints (e.g. unique email).
3. Backend creates a new `platform_users` record.
4. Backend writes an `audit_logs` entry for the creation.
5. System displays the created platform user.

Open Questions:
- Password creation policy (invite flow vs admin-set password) is not specified.
- Whether phone is required/optional depends on schema constraints.

### C) Update a platform user

1. Platform Admin edits allowed fields (e.g. name/phone/status).
2. Backend validates the update and applies allowed changes.
3. Backend writes an `audit_logs` entry including old/new values where relevant.
4. System displays updated values.

Open Question:
- Which fields are editable vs immutable is not explicitly specified.

### D) Activate / deactivate (status change)

1. Platform Admin changes status.
2. Backend validates status value is allowed by schema.
3. Backend writes an audit log.
4. System reflects new status.

Open Question:
- Exact status values and transition rules for `platform_users.status` are not documented here; use database design constraints.

## Alternate flows

- **Duplicate email**: backend rejects create/update with `VALIDATION_ERROR` (or documented code) without leaking internal DB details.
- **Not found**: backend returns `NOT_FOUND` when the user ID does not exist (without sensitive hints).
- **Forbidden**: backend returns `FORBIDDEN` when the actor lacks permission.

## Failure handling

- Follow [[04-api/error-contract|API Error Contract]] categories.
- All errors must be safe and must not leak platform secrets, stack traces, or password/security details.

## Audit requirements

- Creation, updates, and status changes are **sensitive configuration actions** and must create `audit_logs` entries.
- Audit entries must identify actor as `platform_user`.
- Do not store secrets (passwords, hashes) in audit old/new JSON.

## Data impact

- Primary table: `platform_users`
- Audit table: `audit_logs`

## API impact

Open Question:
- Concrete endpoints, request/response shapes, and idempotency rules are not defined yet for this flow. Document the API contract in `04-api` (feature API spec) before implementing endpoints.

## Acceptance criteria (flow-level)

- [ ] Platform users are managed only via platform-authenticated context.
- [ ] No platform user operation is treated as a tenant-staff operation.
- [ ] Password hash is never returned.
- [ ] Create/update/status-change writes business audit log entries.
- [ ] Unauthorized callers receive `FORBIDDEN`.
- [ ] Duplicate email is rejected as validation error.
