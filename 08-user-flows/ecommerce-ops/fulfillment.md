---
title: Fulfillment Flow
owner: Product + QA
status: production-ready
last_reviewed: 2026-04-30
tags: [user-flow, ecommerce-ops, fulfillment]
source: Unified Commerce production scope + database design
---

# Fulfillment Flow

## Actor

Ecommerce Ops.

## Preconditions

- Actor is authenticated where required.
- Tenant context is resolved.
- Required feature and permission are enabled.
- Related module data is configured.

## Main flow

1. Actor starts the workflow from the relevant screen or API action.
2. System validates tenant, role, feature and context.
3. Actor selects or enters required data.
4. System validates business rules and status transition.
5. System persists the workflow result.
6. System updates audit/reporting/notification/sync side effects where needed.
7. Actor receives a clear success or failure result.

## Failure states

| Failure | Expected handling |
|---|---|
| Permission denied | Block action and show safe reason. |
| Feature disabled | Show feature unavailable message. |
| Invalid status | Reject transition and keep current state. |
| Stock/payment conflict | Create controlled conflict or error. |
| Offline unsupported | Queue only if safe; otherwise block. |

## Acceptance checks

- [ ] Correct actor can complete the flow.
- [ ] Incorrect actor cannot complete the flow.
- [ ] Tenant isolation is preserved.
- [ ] Audit and reporting effects are correct.
