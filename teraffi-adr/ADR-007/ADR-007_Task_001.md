# Task: Idempotency Middleware (DB primary + Redis cache) (4-6 hours)
**ADR:** ADR-007  
**Date:** 2025-10-01

## Inputs
- Postgres
- Redis (Premium)
- HTTP layer

## Outputs
- Middleware + E2E test

## Acceptance Criteria
- [ ] 409 on in-progress
- [ ] DB dedupe enforced
- [ ] Backfill cache

## Implementation Details
- Implement DB-first with unique constraint; Redis fast path.

## Files to Create
- packages/api/src/middleware/idempotency.ts
- packages/api/test/idempotency.e2e.spec.ts

## Dependencies
ADR-009
