# Task: Provision Separate Redis Clients (Idem/ReadModels/RateLimit) (3-4 hours)
**ADR:** ADR-009  
**Date:** 2025-10-01

## Inputs
- Three endpoints

## Outputs
- Client factories

## Acceptance Criteria
- [ ] Configs separated
- [ ] Idem=Premium AOF noeviction
- [ ] Health endpoints

## Implementation Details
- Implement factories and health checks.

## Files to Create
- packages/api/src/lib/redis.idem.ts
- packages/readmodels/src/redis.client.ts
- packages/api/src/lib/redis.ratelimit.ts

## Dependencies
ADR-001
