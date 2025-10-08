# Task: Affinity Query Service (5-7 hours)
**ADR:** ADR-011  
**Date:** 2025-10-01

## Inputs
- Neo4j connection
- Query patterns from ADR-011

## Outputs
- Affinity service with typed queries
- REST API endpoints
- Query result caching

## Acceptance Criteria
- [ ] Partnership discovery query implemented
- [ ] Affinity recommendations with filtering
- [ ] Query parameterization (no injection risk)
- [ ] Results cached in Redis (1 hour TTL)
- [ ] p95 latency < 500ms
- [ ] Unit tests with mock Neo4j

## Implementation Details
Implement Cypher queries from ADR-011 "Query Patterns" section. Wrap in TypeScript service with parameter validation. Add Redis caching layer. Expose via REST endpoints.

## Files to Create
- packages/affinity-service/src/neo4j-queries.ts
- packages/affinity-service/src/service.ts
- packages/affinity-service/src/cache.ts
- packages/api/src/routes/affinity.ts
- packages/affinity-service/test/queries.spec.ts

## Dependencies
Task 011-001, Task 011-002, ADR-009 (Redis)