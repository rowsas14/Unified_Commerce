---
title: Data Flow Diagrams
folder: 02-architecture
status: production-ready
last_reviewed: 2026-04-30
tags:
  - architecture
  - diagrams
  - data-flow
  - mermaid
---

# Data Flow Diagrams

## Purpose

This document contains practical data-flow diagrams for the production Unified Commerce platform.

Use these diagrams to understand how modules connect before writing feature specs, APIs, backend services, frontend screens or tests.

## 1. Tenant setup flow

```mermaid
flowchart TD
    A[Platform Admin] --> B[Create Tenant]
    B --> C[Configure Tenant Status, Currency, Timezone, Locale]
    C --> D[Enable Platform Features]
    D --> E[Create Outlets]
    E --> F[Create Roles and Permissions]
    F --> G[Configure Settings and Themes]
    G --> H[Tenant Ready for Catalog, POS and E-Commerce]
```

Related docs:

- [[02-architecture/tenancy-model]]
- [[02-architecture/role-permission-capability-model]]
- [[07-modules/tenant-management/README]]
- [[07-modules/platform-administration/README]]

## 2. Effective access flow

```mermaid
flowchart TD
    Request[User Request] --> Auth{Authenticated?}
    Auth -->|No| Deny[Deny]
    Auth -->|Yes| Tenant{Tenant Active?}
    Tenant -->|No| Deny
    Tenant -->|Yes| Feature{Feature Required?}
    Feature -->|No| Permission{Permission Granted?}
    Feature -->|Yes| Entitled{Tenant Entitled?}
    Entitled -->|No| Deny
    Entitled -->|Yes| Flag{Feature Flag Enabled?}
    Flag -->|No| Deny
    Flag -->|Yes| RoleFeature{Role Feature Allowed?}
    RoleFeature -->|No| Deny
    RoleFeature -->|Yes| Permission
    Permission -->|No| Deny
    Permission -->|Yes| Allow[Allow]
```

Related docs:

- [[02-architecture/role-permission-capability-model]]
- [[04-api/feature-access-api-rules]]
- [[05-backend/feature-access-handling]]

## 3. Product catalog setup flow

```mermaid
flowchart TD
    A[Tenant Admin] --> B[Create Categories]
    B --> C[Create Brands and Suppliers]
    C --> D[Create Product]
    D --> E[Create Variants]
    E --> F[Assign SKU and Barcode]
    F --> G[Assign Attributes and Images]
    G --> H[Assign Tax Class and Return Policy]
    H --> I[Assign Price List Items]
    I --> J[Set POS / E-Commerce Availability]
```

Related docs:

- [[03-data/entities/catalog-entities]]
- [[03-data/entities/pricing-tax-entities]]
- [[07-modules/catalog/README]]

## 4. POS sale flow

```mermaid
sequenceDiagram
    participant Cashier
    participant UI as POS Frontend
    participant API as Backend API
    participant DB as PostgreSQL

    Cashier->>UI: Scan barcode
    UI->>API: Resolve product variant
    API->>DB: Read variant, price, tax, stock
    DB-->>API: Validated item context
    API-->>UI: Item added to cart
    Cashier->>UI: Select payment
    UI->>API: Complete sale request
    API->>DB: Create sales and sale_lines
    API->>DB: Create payments and sale_payment_allocations
    API->>DB: Create stock_movements
    API->>DB: Create receipt
    API-->>UI: Completed sale and receipt payload
```

Related docs:

- [[07-modules/sales-pos/README]]
- [[07-modules/payments/README]]
- [[07-modules/receipts/README]]
- [[08-user-flows/cashier/scan-add-pay]]

## 5. Cash session flow

```mermaid
flowchart TD
    A[Cashier Login] --> B[Open Till Session]
    B --> C[Enter Opening Float]
    C --> D[Perform Sales and Cash Movements]
    D --> E[Close Session]
    E --> F[Enter Counted Cash]
    F --> G[System Calculates Variance]
    G --> H{Variance Requires Approval?}
    H -->|Yes| I[Manager Approves]
    H -->|No| J[Close Session]
    I --> J
```

Related docs:

- [[07-modules/sales-pos/features/tills-sessions/feature-spec]]
- [[07-modules/sales-pos/features/cash-movements/feature-spec]]

## 6. E-Commerce checkout flow

```mermaid
sequenceDiagram
    participant Customer
    participant Storefront
    participant API
    participant DB

    Customer->>Storefront: Add item to cart
    Storefront->>API: Add cart item
    API->>DB: Validate variant, price and stock
    Customer->>Storefront: Checkout
    Storefront->>API: Place order
    API->>DB: Create order and order_items
    API->>DB: Store order address snapshot
    API->>DB: Record payment or pending payment
    API->>DB: Create stock reservation when required
    API-->>Storefront: Order confirmation
```

Related docs:

- [[07-modules/ecommerce-orders/README]]
- [[07-modules/order-workflow/README]]
- [[07-modules/fulfillment-logistics/README]]

## 7. Order fulfillment flow

```mermaid
flowchart TD
    A[Order Confirmed] --> B[Assign Fulfillment Outlet]
    B --> C[Reserve / Pick Stock]
    C --> D{Fulfillment Method}
    D -->|Pickup| E[Ready for Pickup]
    E --> F[Collected]
    D -->|Delivery| G[Packed]
    G --> H[Out for Delivery]
    H --> I[Delivered]
    H --> J[Failed Delivery]
```

Related docs:

- [[07-modules/fulfillment-logistics/README]]
- [[04-api/order-workflow-api-rules]]

## 8. Payment and refund flow

```mermaid
flowchart TD
    A[Payment Request] --> B[Create Payment]
    B --> C{Payment Method}
    C -->|Cash| D[Record Captured Payment]
    C -->|Card/QR/Gateway| E[Record Provider Reference or Gateway Transaction]
    D --> F[Allocate to Sale or Order]
    E --> F
    F --> G[Update Payment Status]
    G --> H{Refund Needed?}
    H -->|Yes| I[Create Refund]
    I --> J[Create Outbound Payment if Applicable]
    J --> K[Allocate Refund]
```

Related docs:

- [[07-modules/payments/README]]
- [[04-api/payment-refund-api-rules]]

## 9. Return and exchange flow

```mermaid
flowchart TD
    A[Find Original Sale or Order] --> B[Select Eligible Items]
    B --> C[Validate Return Policy]
    C --> D{Return or Exchange?}
    D -->|Return| E[Create Return and Return Lines]
    E --> F[Calculate Refund]
    F --> G[Create Refund Allocation]
    G --> H[Post Stock Movement]
    D -->|Exchange| I[Create Return]
    I --> J[Create Exchange]
    J --> K[Add New Exchange Lines]
    K --> L{Difference Direction}
    L -->|Collect| M[Collect Payment]
    L -->|Refund| N[Create Refund]
    L -->|None| O[Complete Exchange]
```

Related docs:

- [[07-modules/returns-exchanges/README]]
- [[08-user-flows/cashier/return-flow]]
- [[08-user-flows/cashier/exchange-flow]]

## 10. Offline sync flow

```mermaid
sequenceDiagram
    participant POS
    participant IndexedDB
    participant API
    participant DB

    POS->>IndexedDB: Store offline sale/payment/receipt
    POS->>API: Reconnect and create sync batch
    API->>DB: Insert offline_sync_batches
    POS->>API: Upload queued items
    API->>DB: Insert offline_sync_items
    API->>DB: Process item transactionally
    alt Accepted
        API->>DB: Write business tables
    else Conflict
        API->>DB: Write offline_sync_conflicts
    else Rejected
        API->>DB: Mark item rejected
    end
```

Related docs:

- [[02-architecture/offline-first-architecture]]
- [[03-data/offline-sync-data-model]]
- [[07-modules/offline-sync/README]]

## 11. Reporting data flow

```mermaid
flowchart LR
    Sales[sales] --> Summaries[Daily Read Models]
    Orders[orders] --> Summaries
    Payments[payments] --> Summaries
    Stock[stock_movements] --> Summaries
    Returns[returns/exchanges] --> Summaries
    Summaries --> Reports[Dashboards and Reports]
    Audit[audit_logs] --> Reports
```

Read models include:

- `daily_sales_summaries`
- `daily_payment_summaries`
- `daily_inventory_summaries`
- `daily_discount_return_summaries`

Related docs:

- [[03-data/entities/reporting-entities]]
- [[07-modules/reporting/README]]

## 12. Receipt generation flow

```mermaid
flowchart TD
    A[Sale / Order / Return / Exchange Completed] --> B[Select Receipt Template]
    B --> C[Build Frozen Receipt Payload]
    C --> D[Store Receipt Record]
    D --> E{Output Type}
    E -->|Thermal Print| F[Print]
    E -->|PDF| G[Generate PDF]
    E -->|Email| H[Send Email]
    F --> I[Write Receipt Print Log]
    G --> I
    H --> I
```

Related docs:

- [[07-modules/receipts/README]]
- [[03-data/entities/receipts-audit-offline-entities]]

## Diagram maintenance checklist

When a workflow changes:

- [ ] Update related feature spec.
- [ ] Update API documentation.
- [ ] Update data/entity documentation.
- [ ] Update this diagram file if flow changed.
- [ ] Update testing workflow cases.
- [ ] Update user flow file.

## Final rule

A diagram is not a replacement for business rules.

Use diagrams to understand workflow order, then read the related module and feature specs for details.
