# Task: Tenant Scope Guard Plugin (impl + unit) (4-6 hours)
**ADR:** ADR-006  
**Date:** 2025-10-01

## Inputs
- Kysely
- Tenant tables
- Request context

## Outputs
- Plugin + 5 unit tests

## Acceptance Criteria
- [ ] Blocks unscoped queries
- [ ] Allows scoped/joins
- [ ] Emits metric

## Implementation Details
- Implement plugin; provide helper and error types.

## Files to Create
- packages/api/src/plugins/TenantScopeGuardPlugin.ts
- packages/api/src/plugins/__tests__/TenantScopeGuardPlugin.test.ts

## Dependencies
ADR-001
