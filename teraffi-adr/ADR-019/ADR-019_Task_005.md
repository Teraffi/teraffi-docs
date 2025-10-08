# Task: Tenant-Scoped Database Client (5-6 hours)
**ADR:** ADR-019  
**Date:** 2025-10-01

## Inputs
- PostgreSQL connection pool
- Tenant context
- RLS policies

## Outputs
- Tenant-scoped database wrapper
- Automatic tenant filtering
- Transaction management

## Acceptance Criteria
- [ ] Wrapper sets app.current_tenant_id before queries
- [ ] Automatic tenant_id injection in WHERE clauses
- [ ] Transaction-scoped tenant context
- [ ] Fallback to context if not explicitly provided
- [ ] Throw error if no tenant context available
- [ ] Integration with existing query builders
- [ ] Unit and integration tests

## Implementation Details
Create database client wrapper that automatically sets tenant context. Use PostgreSQL session variables for RLS. Provide convenience methods that extract tenant from async local storage.

## Files to Create
- packages/database/src/tenant-scoped-db.ts
- packages/database/src/transaction-wrapper.ts
- packages/database/test/tenant-scoped-db.spec.ts

## Dependencies
Tasks 019-002, 019-004