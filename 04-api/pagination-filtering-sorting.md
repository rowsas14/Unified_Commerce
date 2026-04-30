---
title: {title}
owner: API Architecture / Backend Architecture
status: production-ready
last_reviewed: {DATE}
tags: [{tags}]
source_scope: Unified_Commerce_Scope_V1.docx
source_database: Unified_Commerce_Database_Design_final V2.docx
source_backend_architecture: Back end Architecture final.txt
source_frontend_architecture: Frontend archi V1.txt
---
# Pagination, Filtering and Sorting Rules

List APIs must use consistent pagination, filtering, sorting, and search patterns.
This is important for admin screens, POS lookups, catalog management, inventory views, reporting, audit, and e-commerce operations.

## Required reading

- [[00-start-here/README|Start Here]]
- [[01-product/project-scope|Production Project Scope]]
- [[01-product/production-module-catalog|Production Module Catalog]]
- [[02-architecture/system-overview|System Overview]]
- [[02-architecture/tenancy-architecture|Tenancy Architecture]]
- [[02-architecture/role-permission-capability-model|Role Permission Capability Model]]
- [[03-data/database-overview|Database Overview]]
- [[03-data/tenant-consistency-rules|Tenant Consistency Rules]]
- [[09-security-and-compliance/authorization-model|Authorization Model]]
- [[09-security-and-compliance/audit-requirements|Audit Requirements]]


## Standard query parameters

| Parameter | Purpose | Notes |
|---|---|---|
| `page` | Page number | Starts from 1 unless documented otherwise. |
| `pageSize` | Items per page | Must have safe maximum. |
| `search` | Text search | Use for product/customer/order/receipt lookup where supported. |
| `sortBy` | Sort field | Must be allowlisted. |
| `sortDirection` | `asc` or `desc` | Default must be documented. |
| `fromDate` | Start date/time filter | Use ISO format or documented date format. |
| `toDate` | End date/time filter | Use exclusive/inclusive convention consistently. |
| `outletId` | Outlet filter | Must belong to tenant and caller access. |
| `channel` | POS/e-commerce/all filter | Use documented channel values. |
| `status` | Status filter | Must match documented status values. |

## Standard list response

```json
{
  "success": true,
  "data": {
    "items": [],
    "page": 1,
    "pageSize": 25,
    "totalCount": 0
  },
  "meta": {
    "requestId": "...",
    "timestamp": "..."
  }
}
```

## Pagination rules

| Rule | Explanation |
|---|---|
| Always cap `pageSize` | Prevent heavy queries. |
| Default sort must be deterministic | Avoid duplicate/missing records across pages. |
| Use indexes for common filters | Especially tenant/date/status/outlet. |
| Avoid huge unpaged lists | Master data still needs paging if large. |
| Reports may use date windows | Reporting endpoints should restrict date ranges if needed. |

## Common filters by module

| Module | Useful filters |
|---|---|
| Catalog | search, category, brand, status, channel visibility. |
| Inventory | outlet, variant, low stock, date, movement type. |
| POS sales | outlet, till, cashier, business date, status. |
| Payments | method, status, outlet, date, purpose. |
| Orders | order status, payment status, fulfillment status, customer, date. |
| Fulfillment | delivery status, outlet, method, tracking reference. |
| Returns/exchanges | status, source sale/order, outlet, date. |
| Offline sync | device, batch status, item status, conflict type. |
| Reports | date range, outlet, channel, scope type. |
| Audit logs | actor, entity type, action, date, tenant/platform scope. |

## Search rules

Search must be scoped by tenant unless explicitly platform-level.
Searchable fields should be documented per module.

Common searches:

- barcode/SKU/product name;
- sale/order/receipt number;
- customer email/phone/name;
- payment reference;
- delivery tracking number;
- sync batch/device code;
- coupon code.

## Sorting rules

Only allow sorting by documented fields.
Never pass raw sort fields into SQL/ORM dynamically without allowlisting.

| Resource | Common default sort |
|---|---|
| Products | `updated_at desc` or name where appropriate. |
| Sales/orders | `created_at desc` or `business_date desc`. |
| Payments/refunds | `created_at desc`. |
| Stock movements | `occurred_at desc`. |
| Offline sync items | `created_at asc` for processing; `created_at desc` for viewing. |
| Audit logs | `created_at desc`. |

## Tenant and permission filtering

APIs must apply tenant and permission filtering before pagination.
Do not paginate first and then remove unauthorized records in memory.

## Checklist

- [ ] List endpoint has paging.
- [ ] Page size has safe maximum.
- [ ] Filters are documented and tenant-safe.
- [ ] Sort fields are allowlisted.
- [ ] Search fields are documented.
- [ ] Date filters use consistent timezone/business date logic.
- [ ] Query supports expected indexes.
- [ ] Response shape is standard.
