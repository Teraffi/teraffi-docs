# Task: Fraud Review Signal & State (3-4 hours)
**ADR:** ADR-008  
**Date:** 2025-10-01

## Inputs
- Fraud provider stub

## Outputs
- Signal handler

## Acceptance Criteria
- [ ] On-hold state supported
- [ ] Signal transitions to approved/declined
- [ ] Tests

## Implementation Details
- Add fraud_check signal; update state machine.

## Files to Create
- packages/worker/src/workflows/fraud.signal.ts

## Dependencies
ADR-008
