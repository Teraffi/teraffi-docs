# Task: Redis Fallback Flags (Critical Only) (3-4 hours)
**ADR:** ADR-005  
**Date:** 2025-10-01

## Inputs
- Redis client
- Registry

## Outputs
- Fallback evaluator

## Acceptance Criteria
- [ ] TTL & stale warnings
- [ ] Fail-open/closed policy
- [ ] Unit tests

## Implementation Details
- Implement fallback with TTL and warnings; registry-driven.

## Files to Create
- packages/api/src/flags/fallback.ts
- packages/api/test/flags.fallback.spec.ts

## Dependencies
Task 005-001
