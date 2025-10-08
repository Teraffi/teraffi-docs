# Task: Real-Time Indexing (Change Data Capture) (5-6 hours)
**ADR:** ADR-021  
**Date:** 2025-10-01

## Inputs
- Database change events
- Queue infrastructure
- Indexer service

## Outputs
- CDC pipeline
- Real-time index updates
- Event handlers

## Acceptance Criteria
- [ ] Listen for entity create/update/delete events
- [ ] Queue indexing jobs
- [ ] Process updates within 5 minutes
- [ ] Handle reindexing on schema changes
- [ ] Dead letter queue for failures
- [ ] Retry logic (exponential backoff)
- [ ] Monitoring and alerts
- [ ] Integration tests

## Implementation Details
Use database triggers or event sourcing to capture changes. Queue indexing jobs. Process asynchronously. Ensure index stays fresh (< 5 min lag).

## Files to Create
- packages/search/src/cdc/change-handler.ts
- packages/search/src/jobs/index-entity.ts
- packages/search/test/cdc.spec.ts

## Dependencies
Task 021-004, ADR-003 (Queue)