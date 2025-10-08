# Task: Read-Model Cache Strategy & Backfill (3-4 hours)
**ADR:** ADR-009  
**Date:** 2025-10-01

## Inputs
- Redis JSON
- DB access

## Outputs
- Cache module

## Acceptance Criteria
- [ ] Cache-aside pattern
- [ ] Miss backfills
- [ ] Circuit breaker honored

## Implementation Details
- Implement read-through cache with CB when Redis down.

## Files to Create
- packages/readmodels/src/cache.ts

## Dependencies
ADR-006
