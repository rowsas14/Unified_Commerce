---
title: Test Case Template
owner: Documentation Owner
status: production-ready
last_reviewed: 2026-04-30
tags: [template, qa, test-case, unified-commerce]
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
---

# Test Case Template

Use this template for QA test cases under `10-testing-quality/` or feature-level test notes.
Test cases must protect production workflows, financial totals, stock accuracy, tenant isolation and offline sync integrity.

## File location examples

```text
10-testing-quality/payment-refund-test-cases.md
10-testing-quality/offline-sync-test-cases.md
07-modules/sales-pos/features/pos-checkout/test-cases.md
```

## Copy template

```markdown
---
title: <Feature or Flow> Test Cases
owner: QA Team
status: draft
last_reviewed: <YYYY-MM-DD>
tags: [unified-commerce, qa, <module>, <feature>]
module: <module>
feature: <feature>
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
related_docs:
  - `[[07-modules/<module>/features/<feature>/feature-spec]]`
---

# <Feature or Flow> Test Cases

## 1. Test scope

<Explain what is being tested.>

## 2. Source documents

| Document | Used for |
|---|---|
| Feature spec | <rules> |
| Entity reference | <data> |
| API spec | <contracts> |
| User flow | <workflow> |

## 3. Test environment

| Item | Requirement |
|---|---|
| Tenant | <tenant setup> |
| Outlet | <outlet setup> |
| User role | <role/permission> |
| Device/session | <POS device/till/session if applicable> |
| Data | <products/customers/orders/etc.> |

## 4. Test data

| Data item | Value / setup |
|---|---|
| Product variant | <SKU/barcode/price/stock> |
| Customer | <customer state> |
| Payment method | <method> |
| Tax/discount | <rule> |

## 5. Test cases

### TC-001: <Scenario name>

| Field | Value |
|---|---|
| Type | Unit / API / Integration / Frontend / E2E / Regression |
| Priority | Critical / High / Medium / Low |
| Actor | <actor> |
| Preconditions | <preconditions> |

#### Steps

1. <Step 1>
2. <Step 2>
3. <Step 3>

#### Expected result

- <Expected result 1>
- <Expected result 2>
- <Expected result 3>

#### Data assertions

| Entity/table | Expected assertion |
|---|---|
| `<table>` | <assertion> |

#### Audit assertions

- [ ] <audit event expected or not applicable>

## 6. Negative test cases

| Scenario | Expected result |
|---|---|
| <invalid action> | <blocked/error behavior> |

## 7. Permission and feature access tests

| Scenario | Expected result |
|---|---|
| Feature disabled | Action blocked. |
| Permission missing | 403 or UI blocked and backend rejects. |
| Wrong tenant | Data not visible/action rejected. |

## 8. Regression checklist

- [ ] Tenant isolation still works.
- [ ] Totals are correct.
- [ ] Stock is correct.
- [ ] Audit is written.
- [ ] Offline behavior is correct where applicable.
- [ ] Reports remain correct.
```

## Test category rules

| Test category | Use for |
|---|---|
| Unit | Domain/application validation and calculation rules. |
| API | Endpoint request/response, auth, permission, validation. |
| Integration | Database transaction, repository, cross-module workflow. |
| Frontend | Component, form, state and UI behavior. |
| E2E | Full business workflow through UI/API. |
| Regression | Critical no-break workflows after changes. |

## Mandatory production risk tests

Add these tests where relevant:

| Risk area | Required test |
|---|---|
| Tenant isolation | User from Tenant A cannot read/write Tenant B data. |
| RBAC | User without permission cannot perform protected action. |
| Feature access | Tenant-disabled feature cannot execute backend operation. |
| Payment | Duplicate retry does not create duplicate payment. |
| Refund | Refund cannot exceed captured amount. |
| Stock | Stock movement creates correct balance/projection. |
| Offline sync | Duplicate client transaction is rejected or safely reused. |
| Receipt | Reprint requires permission and audit. |
| Tax/discount | Calculations are consistent in sale/order/refund/report. |
| Reporting | Reports match source transaction data. |

## POS-specific test data

For POS features, include:

- Active tenant.
- Active outlet.
- Assigned POS device.
- Active till.
- Open till session.
- Cashier role.
- Product variant with barcode.
- Inventory balance at outlet.
- Payment method enabled.
- Receipt template active.

## E-Commerce-specific test data

For e-commerce features, include:

- Tenant storefront enabled.
- Online-visible product.
- Customer or guest token.
- Cart with valid line items.
- Delivery/pickup method.
- Payment method.
- Address snapshot.
- Order status setup.

## Offline sync test data

For offline sync features, include:

- Registered POS device.
- Cached products/prices/tax settings.
- Local client transaction ID.
- Offline sale payload.
- Offline payment payload.
- Sync batch.
- Conflict scenario, if testing conflict.

## Example critical test: POS sale completion

| Field | Value |
|---|---|
| Type | E2E + integration |
| Priority | Critical |
| Actor | Cashier |

Steps:

1. Open active till session.
2. Scan product barcode.
3. Apply allowed discount, if test requires.
4. Complete cash payment.
5. Print receipt.

Expected:

- Sale status becomes completed.
- Sale lines are stored.
- Payment is stored and allocated.
- Stock movement is created.
- Receipt is generated.
- Cash expected amount is updated.
- Report source data is available.

## Example negative test: refund exceeds amount

Steps:

1. Create completed sale with captured payment.
2. Attempt refund greater than captured amount.

Expected:

- API rejects with business validation error.
- No refund row is completed.
- No outbound payment is created.
- Audit records failed/sensitive attempt only if policy requires.

## Acceptance quality rules

A test case is not complete unless expected results are measurable.
Avoid vague results like “works correctly”.

Correct:

```text
payments.captured_amount remains 100.00 and refunds total cannot exceed 100.00.
```

Wrong:

```text
Refund works fine.
```

## QA completion checklist

- [ ] Test has clear preconditions.
- [ ] Test data is defined.
- [ ] Steps are reproducible.
- [ ] Expected results are measurable.
- [ ] Data assertions are listed.
- [ ] Permission/feature access is tested where relevant.
- [ ] Negative path is tested.
- [ ] Audit behavior is tested where relevant.
- [ ] Regression risk is marked.
- [ ] Related docs are linked.
