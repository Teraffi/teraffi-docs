# Task: Row-Level Security Implementation (3-4 hours)
**ADR:** ADR-018  
**Date:** 2025-10-01

## Inputs
- Security requirements
- Tenant isolation needs
- User roles

## Outputs
- Row-level security policies
- Query filters
- Access control

## Acceptance Criteria
- [ ] Filter data by tenant_id
- [ ] Filter by user's accessible entities
- [ ] Enforce in all dashboard queries
- [ ] Enforce in API queries
- [ ] Cannot bypass filters via direct database access
- [ ] Performance impact minimal (<10% overhead)
- [ ] Audit logging of access
- [ ] Documentation

## Implementation Details
Implement row-level security in ClickHouse and/or application layer. Add tenant_id filters to all queries. Create user context that specifies accessible data. Enforce in Metabase and API.

## Files to Create
- packages/analytics-api/src/middleware/row-level-security.ts
- analytics/clickhouse/policies/tenant_isolation.sql
- docs/security/analytics_access_control.md

## Dependencies
Task 018-005, Task 018-009