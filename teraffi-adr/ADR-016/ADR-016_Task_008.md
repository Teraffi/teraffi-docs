# Task: Neo4j Trend Integration (4-5 hours)
**ADR:** ADR-016  
**Date:** 2025-10-01

## Inputs
- Extracted trends
- Momentum metrics
- Neo4j connection

## Outputs
- Trend updater service
- Trend node creation/update
- Demographic relationships

## Acceptance Criteria
- [ ] Create/update Trend nodes in Neo4j
- [ ] Store momentum, velocity, lifecycle stage
- [ ] Track emergence and peak dates
- [ ] Create APPEALS_TO relationships with Demographics
- [ ] Handle duplicate trends (MERGE not CREATE)
- [ ] Archive declining trends (>90 days)
- [ ] Integration tests

## Implementation Details
Use MERGE to upsert Trend nodes. Update momentum metrics on each run. Create relationships to demographic nodes based on appeal. Archive old declining trends to reduce graph size.

## Files to Create
- packages/social-listening/src/trend-updater.ts
- packages/social-listening/src/archival/trend-archiver.ts
- packages/social-listening/test/trend-updater.spec.ts

## Dependencies
Task 016-005, Task 016-007, ADR-011 (Neo4j)