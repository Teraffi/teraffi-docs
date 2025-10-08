# Task: PostgreSQL → Neo4j Sync Worker (6-8 hours)
**ADR:** ADR-011  
**Date:** 2025-10-01

## Inputs
- Service Bus subscription
- Neo4j connection
- Outbox events schema

## Outputs
- Sync worker service
- Event handlers for member/partnership events
- Error handling and retry logic

## Acceptance Criteria
- [ ] Consumes member.created, member.profile_updated events
- [ ] Creates/updates Member nodes in Neo4j
- [ ] Handles partnership.created → PARTNERS_WITH relationship
- [ ] Idempotent (duplicate events don't cause errors)
- [ ] Dead letter queue for failed syncs
- [ ] Metrics emitted (sync lag, success rate)

## Implementation Details
Subscribe to Service Bus topics. Map PostgreSQL events to Cypher MERGE statements. Use pg_synced_at timestamp for reconciliation. Implement exponential backoff for retries.

## Files to Create
- packages/graph-sync/src/worker.ts
- packages/graph-sync/src/handlers/member.ts
- packages/graph-sync/src/handlers/partnership.ts
- packages/graph-sync/test/sync.spec.ts

## Dependencies
Task 011-001, ADR-003 (Outbox), ADR-009 (Redis for deduplication)