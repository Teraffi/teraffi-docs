# Task: PostgreSQL Row-Level Security (RLS) Policies (5-6 hours)
**ADR:** ADR-019  
**Date:** 2025-10-01

## Inputs
- All tables with tenant_id
- RLS policy requirements

## Outputs
- RLS policies on all tables
- Database session configuration
- Testing procedures

## Acceptance Criteria
- [ ] Enable RLS on all tenant-scoped tables
- [ ] Create tenant_isolation_policy for each table
- [ ] Policy filters by current_setting('app.current_tenant_id')
- [ ] Set tenant context at transaction start
- [ ] Test policies prevent cross-tenant access
- [ ] Performance impact <5%
- [ ] Documentation

## Implementation Details
Enable PostgreSQL RLS on all tables. Create policies that filter by app.current_tenant_id session variable. Wrap database transactions to set tenant context before queries execute.

## Files to Create
- db/migrations/XXX_enable_rls_policies.sql
- packages/database/src/rls-config.ts
- packages/database/test/rls-enforcement.spec.ts
- docs/security/row_level_security.md

## Dependencies
Task 019-001