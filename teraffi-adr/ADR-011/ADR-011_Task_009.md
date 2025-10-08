# Task: Comprehensive Neo4j Test Suite (5-7 hours)
**ADR:** ADR-011  
**Date:** 2025-10-01

## Inputs
- All Neo4j services
- Test data fixtures

## Outputs
- Integration tests
- Query performance tests
- Sync worker tests

## Acceptance Criteria
- [ ] Schema validation tests
- [ ] Query correctness tests (affinity, trends, partnerships)
- [ ] Sync worker idempotency tests
- [ ] Performance benchmarks (query latency targets)
- [ ] Error handling tests (Neo4j unavailable, timeout)
- [ ] CI green

## Implementation Details
Create comprehensive test suite covering all query patterns. Use Neo4j testcontainer for integration tests. Mock Neo4j for unit tests. Benchmark query performance against targets.

## Files to Create
- packages/graph/test/schema.spec.ts
- packages/affinity-service/test/integration.spec.ts
- packages/graph-sync/test/worker.spec.ts
- packages/graph/test/performance.spec.ts

## Dependencies
Tasks 011-001 through 011-006