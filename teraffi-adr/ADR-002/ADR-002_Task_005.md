# Task: Fulfillment Workflow (Capture on Ship) (6-8 hours)
**ADR:** ADR-002  
**Date:** 2025-10-01

## Inputs
- WMS signal stub
- Payment capture

## Outputs
- Workflow + activity

## Acceptance Criteria
- [ ] Capture on ship
- [ ] Retries [1,5,15]
- [ ] Hold/cancel policy

## Implementation Details
- Implement capture with error classes and policy handling.

## Files to Create
- packages/worker/src/workflows/fulfillment.workflow.ts
- packages/worker/src/activities/paymentCapture.activity.ts

## Dependencies
Task 002-004
