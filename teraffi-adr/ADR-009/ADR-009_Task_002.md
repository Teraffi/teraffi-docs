# Task: Chaos Toggle & Drill Scripts (Idempotency Safety) (3-4 hours)
**ADR:** ADR-009  
**Date:** 2025-10-01

## Inputs
- Config flag
- Drill scripts

## Outputs
- Drill scripts + doc

## Acceptance Criteria
- [ ] Writes pause safely
- [ ] No duplicate charges
- [ ] Alerts fire

## Implementation Details
- Add toggle to simulate failure and drill docs.

## Files to Create
- docs/runbooks/redis_drill.md
- packages/ops/src/drills/redis.ts

## Dependencies
Task 009-001
