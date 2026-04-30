---
title: Security Architecture
folder: 02-architecture
status: production-ready
last_reviewed: 2026-04-30
tags:
  - security
  - authorization
  - audit
  - data-protection
---

# Security Architecture

## Purpose

This document defines the architecture-level security rules for the Unified Commerce platform.

Detailed security policies live in [[09-security-and-compliance/README]].

## Security goals

The system must protect:

- Tenant data isolation.
- Staff and customer identities.
- Payments and refunds.
- Stock and financial records.
- Offline POS data.
- Feature configuration.
- Audit integrity.

## Identity boundaries

| Identity type | Table(s) | Scope |
|---|---|---|
| Platform admin | `platform_users` | Platform |
| Tenant staff | `users` | Tenant |
| Customer | `customers`, `customer_auth_accounts`, `customer_auth_identities` | Tenant storefront |
| POS device | `pos_devices` | Tenant + outlet + till |

These identities must not be merged into one ambiguous account model.

## Authentication architecture

Authentication must establish:

- Actor identity.
- Actor type.
- Tenant context where applicable.
- Outlet/device/session context where applicable.
- Account status.

Authentication alone does not mean authorization.

## Authorization architecture

Authorization uses layered access.

See [[02-architecture/role-permission-capability-model]].

Backend must validate:

- Tenant active status.
- User active status.
- Tenant/outlet assignment.
- Feature entitlement.
- Feature flag.
- Role feature assignment.
- Permission.
- Business state.

## Backend authority rule

Frontend hiding is not security.

Backend must protect every sensitive operation even if:

- User manipulates the browser.
- User calls API directly.
- Cached frontend state is stale.
- Offline data is uploaded later.

## Tenant isolation security

Cross-tenant access must be blocked in:

- API route handling.
- Application services.
- Repository queries.
- Database constraints where possible.
- Tests.

Example:

Do not fetch order by `order_id` only.

Fetch by `tenant_id` and `order_id`.

## Sensitive actions

Sensitive actions require permission and audit.

Examples:

- Refund approval.
- Sale void.
- Receipt reprint.
- Discount approval.
- Stock adjustment posting.
- Cash variance approval.
- Tenant feature entitlement change.
- Role/permission change.
- Offline conflict resolution.
- Payment provider configuration change.

## Audit architecture

Business audit uses `audit_logs`.

Sync technical audit uses `offline_sync_audit_logs`.

Audit records should include:

- Actor.
- Actor type.
- Tenant.
- Device where applicable.
- Entity type.
- Entity ID.
- Action.
- Old values where useful.
- New values where useful.
- IP/user agent where available.
- Timestamp.

## OTP security architecture

OTP uses:

- `otp_channels`
- `otp_purposes`
- `otp_verifications`

Rules:

- Store only hashed OTP values.
- Track attempt count.
- Track resend count.
- Track last attempt time.
- Use `blocked_until` after abuse threshold.
- Store destination and metadata for monitoring.

## Payment security architecture

Payment security rules:

- Do not store card data.
- Do not store provider private keys in plain JSON.
- Use `secret_ref` for provider secrets.
- Store provider events in `payment_transactions`.
- Use idempotency keys for duplicate protection.
- Refunds must not exceed captured amounts.

## Offline data security

Offline POS data is stored locally in IndexedDB.

Risk must be reduced by:

- Storing minimum required operational data.
- Avoiding secrets in offline storage.
- Binding data to device/tenant/outlet.
- Clearing or invalidating local data when device assignment changes.
- Revalidating all queued data during sync.

See [[02-architecture/offline-first-architecture]].

## Device security architecture

POS devices are registered in `pos_devices`.

Device security should check:

- Device status is active.
- Device belongs to tenant.
- Device belongs to outlet.
- Device belongs to till.
- Device fingerprint matches where supported.
- Blocked devices cannot sync or bill.

## Configuration security

Configuration changes can affect business behavior.

Sensitive configuration includes:

- Feature entitlements.
- Feature flags.
- Tenant settings.
- UI themes.
- Payment provider configuration.
- Receipt templates.
- Discount policies.
- Tax settings.

These changes should be permission-controlled and audited.

## Status transition security

Users must not freely set status fields.

Status transitions must be validated.

Examples:

- Delivered order should not move back to processing without controlled correction.
- Captured payment should not become pending.
- Closed till session should not accept normal sales unless offline sync rule permits delayed posting.

## Data exposure rules

API responses must not expose:

- Password hashes.
- OTP hashes.
- Provider secrets.
- Internal security metadata not needed by UI.
- Other tenant IDs/data.
- Raw provider payloads to unauthorized users.

## Security testing checklist

- [ ] Tenant A cannot read Tenant B data.
- [ ] User without permission cannot call protected API.
- [ ] Disabled feature cannot be used through API.
- [ ] Blocked POS device cannot sync.
- [ ] Refund amount cannot exceed original captured amount.
- [ ] Receipt reprint is audited.
- [ ] OTP cannot be brute forced without blocking.
- [ ] Offline sync duplicate is rejected/idempotent.
- [ ] Provider secrets are not exposed.

## Related docs

- [[09-security-and-compliance/authentication-model]]
- [[09-security-and-compliance/authorization-model]]
- [[09-security-and-compliance/password-and-otp-rules]]
- [[09-security-and-compliance/audit-requirements]]
- [[09-security-and-compliance/data-isolation-controls]]
- [[02-architecture/role-permission-capability-model]]

## Final rule

Security must be enforced server-side and verified by tests.

Frontend UX improves usability, but it is not the security boundary.
