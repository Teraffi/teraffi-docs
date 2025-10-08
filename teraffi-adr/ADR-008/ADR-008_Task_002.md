# Task: Fulfillment Workflow (Capture on Ship + Policy) (6-8 hours)
**ADR:** ADR-008  
**Date:** 2025-10-01

## Inputs
- WMS signal
- Payment capture

## Outputs
- Workflow + activity

## Acceptance Criteria
- [ ] Capture with retries
- [ ] Hold/cancel on final fail
- [ ] Inventory release on cancel

## Implementation Details
- Implement error classes and policy.

## Files to Create
- packages/worker/src/workflows/fulfillment.workflow.ts

## Dependencies
Task 008-001
