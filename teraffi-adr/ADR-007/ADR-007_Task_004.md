# Task: Idempotency Redis Client (Premium AOF) (2-3 hours)
**ADR:** ADR-007  
**Date:** 2025-10-01

## Inputs
- Redis URL

## Outputs
- Client factory

## Acceptance Criteria
- [ ] AOF enabled
- [ ] noeviction set
- [ ] Healthcheck

## Implementation Details
- Add dedicated client for idempotency; health endpoint.

## Files to Create
- packages/api/src/lib/redis.idem.ts

## Dependencies
ADR-009
