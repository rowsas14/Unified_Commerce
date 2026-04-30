---
title: Data Import and AI-Assisted Onboarding Data Notes
folder: 03-data/entities
status: production-ready
owner: Data Import / AI Onboarding
tags: [data-import, ai-onboarding, schema-gap]
---

# Data Import and AI-Assisted Onboarding Data Notes

The production scope includes CSV/Excel import and AI-assisted onboarding, but the uploaded database design does not define dedicated import or AI staging tables. This file prevents accidental invention of schema during implementation.

Based on the uploaded production scope, production database design, backend architecture, frontend architecture, and current Unified Commerce 2nd Brain structure.

---

## Table map

No dedicated tables are present in the uploaded database design for this scope area. See [[../required-schema-extensions]].

---

## Relationship diagram

```mermaid
flowchart LR
    scope[Scope requirement] --> gap[Schema gap]
```

---

## Production data rules

- CSV/Excel product and customer import is in the production scope.
- AI-assisted PDF/image extraction is in scope as an assisted onboarding capability.
- The uploaded database design does not include dedicated import job, parsed row, AI extraction, or review decision tables.
- Developers must not create ad-hoc import/AI tables without an approved database design update.
- Accepted imported data must ultimately land in normal source tables such as products, product_variants, customers, suppliers, inventory documents, and audit logs.

---

## Implementation checklist

- [ ] Tenant ownership and parent-child tenant consistency are enforced.
- [ ] All FK relationships are mapped in EF Core and validated at service boundary.
- [ ] Unique constraints and partial unique indexes are implemented where documented.
- [ ] Status values are validated before writes.
- [ ] Audit behavior is defined for sensitive changes.
- [ ] Offline sync impact is checked if POS/device/offline records are involved.
- [ ] Reporting impact is understood before changing source tables.
- [ ] Related API, backend, frontend, module, and test docs are updated.

---

## Related files

- [[../required-schema-extensions]]
- [[catalog-entities]]
- [[customer-ecommerce-entities]]
- [[../../07-modules/data-import-ai/README]]
