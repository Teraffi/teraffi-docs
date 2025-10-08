# Task: Triple Freshness Management (3-4 hours)
**ADR:** ADR-015  
**Date:** 2025-10-01

## Inputs
- Neo4j graph data
- Last updated timestamps
- Refresh queue

## Outputs
- Freshness manager service
- Auto-refresh job (30-90 day TTL)
- Stale triple detection

## Acceptance Criteria
- [ ] Identify triples older than 90 days
- [ ] Enqueue for refresh via background job
- [ ] Re-fetch public data for stale entities
- [ ] Update graph with fresh triples
- [ ] Daily batch job or continuous monitoring
- [ ] Metrics on freshness distribution
- [ ] Unit tests

## Implementation Details
Query Neo4j for entities with last_updated > 90 days. Enqueue refresh jobs. Re-run public data aggregation and update graph. Track freshness metrics.

## Files to Create
- packages/affinity-engine/src/freshness-manager.ts
- packages/affinity-engine/src/jobs/refresh-triples.ts
- packages/affinity-engine/test/freshness.spec.ts

## Dependencies
Task 015-002, Task 015-003