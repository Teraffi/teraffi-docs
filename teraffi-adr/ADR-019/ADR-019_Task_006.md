# Task: Neo4j Tenant Isolation (4-5 hours)
**ADR:** ADR-019  
**Date:** 2025-10-01

## Inputs
- Neo4j graph database
- Tenant context
- Query patterns

## Outputs
- Tenant-scoped Neo4j wrapper
- Automatic tenant filtering
- Query injection

## Acceptance Criteria
- [ ] Add tenant_id property to all nodes
- [ ] Wrapper injects tenant_id filter in MATCH clauses
- [ ] Support for both read and write queries
- [ ] Validate all queries include tenant filter
- [ ] Performance impact minimal
- [ ] Unit tests
- [ ] Documentation

## Implementation Details
Create Neo4j wrapper that automatically adds {tenant_id: $tenant_id} filters to MATCH clauses. Parse queries to inject tenant filters. Provide session management with tenant context.

## Files to Create
- packages/graph/src/tenant-scoped-neo4j.ts
- packages/graph/src/query-injector.ts
- packages/graph/test/tenant-scoped-neo4j.spec.ts

## Dependencies
Task 019-002, ADR-011 (Neo4j)