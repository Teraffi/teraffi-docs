# Task: Relay Upgrade (Phase 2: LISTEN/NOTIFY) (3-4 hours)
**ADR:** ADR-003  
**Date:** 2025-10-01

## Inputs
- pg notify

## Outputs
- Notify subscriber

## Acceptance Criteria
- [ ] Immediate wakeups
- [ ] Storm backoff
- [ ] Tests

## Implementation Details
- Add NOTIFY on insert; subscriber wakes relay workers.

## Files to Create
- packages/ops/src/outboxNotify.ts

## Dependencies
Task 003-002
