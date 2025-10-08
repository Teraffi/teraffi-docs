# Task: Tenant Admin UI (6-8 hours)
**ADR:** ADR-019  
**Date:** 2025-10-01

## Inputs
- Tenant management requirements
- Admin user needs

## Outputs
- Admin dashboard
- Tenant CRUD interface
- Usage monitoring

## Acceptance Criteria
- [ ] List all tenants
- [ ] View tenant details (usage, quotas)
- [ ] Create new tenant
- [ ] Edit tenant (tier, quotas, features)
- [ ] Deactivate/reactivate tenant
- [ ] View usage metrics per tenant
- [ ] Trigger data deletion
- [ ] Audit log viewer
- [ ] E2E tests

## Implementation Details
Build admin UI for tenant management. Display tenant list with search/filter. Show usage vs quotas. Allow tier changes and configuration updates. Provide audit log viewer.

## Files to Create
- apps/admin/src/pages/tenants/index.tsx
- apps/admin/src/pages/tenants/[id].tsx
- apps/admin/src/components/TenantForm.tsx
- apps/admin/test/tenants.e2e.spec.ts

## Dependencies
Task 019-009