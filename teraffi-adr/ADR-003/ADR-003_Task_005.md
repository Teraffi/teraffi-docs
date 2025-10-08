# Task: Read Model Publisher Wiring (4-6 hours)
**ADR:** ADR-003  
**Date:** 2025-10-01

## Inputs
- Outbox events
- Redis JSON

## Outputs
- Publisher service

## Acceptance Criteria
- [ ] Variant patch vs rebuild
- [ ] Circuit breaker 100/min
- [ ] Pointer guard

## Implementation Details
- Build publisher honoring granularity rules; version pointer update guard.

## Files to Create
- packages/readmodels/src/publisher.ts

## Dependencies
ADR-006, ADR-010
