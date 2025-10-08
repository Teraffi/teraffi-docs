# Task: Redis Tenant Key Scoping (3-4 hours)
**ADR:** ADR-019  
**Date:** 2025-10-01

## Inputs
- Redis client
- Tenant context

## Outputs
- Tenant-scoped Redis wrapper
- Key prefixing
- Bulk deletion support

## Acceptance Criteria
- [ ] All keys prefixed with tenant:{tenant_id}:
- [ ] Get/Set/Delete operations scoped
- [ ] Support for pattern-based operations
- [ ] Bulk deletion for tenant (GDPR)
- [ ] Transparent to calling code
- [ ] Unit tests
- [ ] Documentation

## Implementation Details
Wrap Redis client to automatically prefix all keys with tenant identifier. Provide methods for tenant-scoped operations. Support deletion of all keys for a tenant (GDPR compliance).

## Files to Create
- packages/cache/src/tenant-scoped-redis.ts
- packages/cache/test/tenant-scoped-redis.spec.ts

## Dependencies
Task 019-002, ADR-009 (Redis)