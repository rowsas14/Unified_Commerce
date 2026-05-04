---
title: Staff User Setup Flow
owner: Product + QA
status: draft
last_reviewed: 2026-05-01
tags: [user-flow, tenant-admin, identity-access, staff-users]
source: Unified Commerce production scope + database design
related_docs:
  - [[07-modules/identity-access/README|Identity and Access Module]]
  - [[07-modules/identity-access/features/staff-users/feature-spec|Staff Users Feature Spec]]
  - [[02-architecture/tenancy-architecture|Tenancy Architecture]]
  - [[02-architecture/role-permission-capability-model|Role Permission Capability Model]]
  - [[04-api/auth-and-authorization|Auth and Authorization API Rules]]
  - [[04-api/tenant-context-api-rules|Tenant Context API Rules]]
  - [[04-api/error-contract|API Error Contract]]
  - [[09-security-and-compliance/authentication-model|Authentication Model]]
  - [[09-security-and-compliance/authorization-model|Authorization Model]]
  - [[09-security-and-compliance/audit-requirements|Audit Requirements]]
---

# Staff User Setup Flow

## Actor

Tenant Admin.

## Purpose

Create and manage **tenant-bound staff users** (`users`) for a tenant, including account status and identity fields (email/phone), while enforcing tenant isolation, RBAC, feature access, and audit.

## Preconditions

- Actor is authenticated as a **tenant staff user** (not a platform user, not a customer).
- Actor has required permission(s) to create/update staff users.
  - Open Question: exact permission code(s) for staff user creation/update are not listed in the current feature spec; use the permission catalog + role-permission mapping when finalized.
- Tenant is active.

## Authentication (JWT)

- Requests in this flow are authenticated using a **JWT access token**.
- Client sends:

```http
Authorization: Bearer <access_token>
```

- Tenant context must be resolved from authenticated context in production.
  - Open Question: exact claim names for `tenant_id` and `user_id` must match the final auth token contract.

## Tenant isolation rules

- Every `users` operation is **tenant-scoped**:
  - create: `users.tenant_id = current tenant`
  - read/update: `WHERE tenant_id = current tenant AND id = user_id`
- Duplicate prevention (email/phone) is tenant-scoped using `normalized_email` / `normalized_phone` uniqueness.

## Happy path

### A) View staff users

1. Tenant Admin opens Staff Users screen.
2. System requests staff user list for current tenant.
3. Backend returns staff users with safe fields only.
   - Must not return `password_hash`.

Open Question:
- Filtering/search/pagination requirements are not specified yet.

### B) Create staff user

1. Tenant Admin enters staff user details (full name, email and/or phone, initial password, status).
2. Backend validates:
   - required fields (e.g., full name, password),
   - tenant context,
   - uniqueness of normalized email/phone within tenant,
   - allowed status values per schema.
3. Backend creates `users` row.
4. Backend writes `audit_logs` entry with actor (`actor_user_id`) and tenant (`tenant_id`).
5. System shows created staff user.

### C) Update staff user profile

1. Tenant Admin edits allowed fields (name, email/phone, reset password where allowed).
2. Backend validates:
   - tenant ownership,
   - duplicate normalized email/phone,
   - status constraints.
3. Backend updates `users` record.
4. Backend writes `audit_logs` entry including old/new values (no secrets).

### D) Activate / deactivate staff user

1. Tenant Admin changes staff user status.
2. Backend validates allowed status values.
3. Backend writes `audit_logs` entry.
4. System reflects status.

## Alternate / error flows

- **Duplicate email/phone**: reject with `VALIDATION_ERROR` / `CONFLICT` according to [[04-api/error-contract]].
- **Cross-tenant access attempt**: return safe `NOT_FOUND` or `FORBIDDEN` response per security policy.
- **Permission missing**: return `FORBIDDEN`.

## Audit requirements

- Create/update/status-change are sensitive identity actions → must write `audit_logs`.
- Audit must not store secrets:
  - never include password, password hash, tokens.

## Data impact

- Primary table: `users`
- Audit table: `audit_logs`

## API impact

Open Question:
- Concrete endpoint list and request/response contract should be documented in `04-api` feature API spec before finalizing production routes.

## Acceptance criteria

- [ ] Staff users are always tenant-scoped.
- [ ] Password hash is never returned.
- [ ] Duplicate normalized email/phone within tenant is rejected.
- [ ] Cross-tenant access is rejected.
- [ ] Create/update/status-change generates `audit_logs` entries.
- [ ] Requests require valid JWT Bearer token.
