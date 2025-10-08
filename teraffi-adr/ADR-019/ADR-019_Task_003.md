# Task: Tenant Registry Service (4-5 hours)
**ADR:** ADR-019  
**Date:** 2025-10-01

## Inputs
- Tenants table
- Redis for caching
- Tier definitions

## Outputs
- Tenant registry service
- CRUD operations
- Cache layer

## Acceptance Criteria
- [ ] Get tenant by ID
- [ ] Create new tenant
- [ ] Update tenant metadata
- [ ] Deactivate tenant
- [ ] Cache tenant data (5 min TTL)
- [ ] Support tier-based features and quotas
- [ ] Return null for non-existent tenants
- [ ] Unit tests

## Implementation Details
Build service to manage tenant metadata. Cache frequently accessed tenant data in Redis. Support CRUD operations. Define features and quotas per tier (free, professional, enterprise).

## Files to Create
- packages/tenant-management/src/tenant-registry.ts
- packages/tenant-management/src/types/tenant.ts
- packages/tenant-management/test/tenant-registry.spec.ts

## Dependencies
Task 019-001, ADR-009 (Redis)