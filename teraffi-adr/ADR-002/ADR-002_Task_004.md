# Task: Checkout Workflow (Authorize Only + DB SoT) (6-8 hours)
**ADR:** ADR-002  
**Date:** 2025-10-01

## Inputs
- Payment adapter stub
- DB repo

## Outputs
- Workflow + activities

## Acceptance Criteria
- [ ] Authorize only
- [ ] Order pending created
- [ ] Reservation placed
- [ ] Idempotent activities

## Implementation Details
- Implement workflow storing correlation IDs; activities query DB first.

## Files to Create
- packages/worker/src/workflows/checkout.workflow.ts
- packages/worker/src/activities/paymentAuthorize.activity.ts

## Dependencies
ADR-007 unique keys
