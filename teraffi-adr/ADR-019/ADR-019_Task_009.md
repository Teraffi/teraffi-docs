# Task: Tenant Provisioning Service (5-6 hours)
**ADR:** ADR-019  
**Date:** 2025-10-01

## Inputs
- Tenant creation requirements
- Tier definitions
- Email service

## Outputs
- Provisioning service
- Deprovisioning service
- Default data seeding

## Acceptance Criteria
- [ ] Create tenant with admin user
- [ ] Set up tier-based features and quotas
- [ ] Seed default configuration
- [ ] Send welcome email
- [ ] Deactivate tenant
- [ ] Schedule data deletion (30 day grace)
- [ ] Audit logging
- [ ] Unit tests

## Implementation Details
Build service to provision new tenants. Create tenant record, admin user, default configuration. Implement deprovisioning with grace period. Support tier upgrades/downgrades.

## Files to Create
- packages/tenant-management/src/provisioning.ts
- packages/tenant-management/src/tier-config.ts
- packages/tenant-management/test/provisioning.spec.ts

## Dependencies
Task 019-003