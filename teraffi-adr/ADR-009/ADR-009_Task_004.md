# Task: Rate Limiter Middleware (token bucket) (3-4 hours)
**ADR:** ADR-009  
**Date:** 2025-10-01

## Inputs
- Redis client

## Outputs
- Middleware + config

## Acceptance Criteria
- [ ] Per-tenant/route limits
- [ ] Burst + refill
- [ ] E2E tests

## Implementation Details
- Implement distributed token bucket; integrate into API.

## Files to Create
- packages/api/src/middleware/ratelimit.ts
- packages/api/test/ratelimit.e2e.spec.ts

## Dependencies
ADR-004 publishable keys
