# Task: Search Performance Optimization (4-5 hours)
**ADR:** ADR-021  
**Date:** 2025-10-01

## Inputs
- Search query patterns
- Performance metrics

## Outputs
- Query optimizations
- Caching layer
- Index tuning

## Acceptance Criteria
- [ ] Cache popular queries (Redis, 5 min TTL)
- [ ] Optimize Elasticsearch queries
- [ ] Index refresh interval tuning
- [ ] Shard allocation optimization
- [ ] Connection pooling
- [ ] Query profiling and slow query logging
- [ ] Performance <200ms (p95)
- [ ] Load testing (1000+ QPS)

## Implementation Details
Implement caching for popular queries. Optimize Elasticsearch index settings. Profile slow queries. Tune shard count and replica settings. Load test and optimize.

## Files to Create
- packages/search/src/cache/query-cache.ts
- packages/search/src/performance/profiler.ts
- packages/search/test/load-test.spec.ts

## Dependencies
Task 021-015