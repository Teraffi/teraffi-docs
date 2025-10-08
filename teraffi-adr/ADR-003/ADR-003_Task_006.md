# Task: Outbox Phase Transition Automation (3-4 hours)
**ADR:** ADR-003  
**Date:** 2025-10-01

## Inputs
- Metrics queries

## Outputs
- Controller

## Acceptance Criteria
- [ ] Triggers P1→P2 at thresholds
- [ ] Alerts created
- [ ] Tests

## Implementation Details
- Evaluate lag/EPS; flip relay mode and scale workers.

## Files to Create
- packages/ops/src/outboxController.ts
- packages/ops/test/outboxController.spec.ts

## Dependencies
Task 003-002
