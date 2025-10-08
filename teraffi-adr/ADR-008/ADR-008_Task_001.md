# Task: Checkout Workflow (Authorize + Order Pending + Reserve) (6-8 hours)
**ADR:** ADR-008  
**Date:** 2025-10-01

## Inputs
- Temporal
- Payment stub
- DB repos

## Outputs
- Workflow + activities

## Acceptance Criteria
- [ ] Authorize only
- [ ] Order pending created
- [ ] Reservation placed

## Implementation Details
- Implement idempotent activities with SoT reads; correlation IDs only.

## Files to Create
- packages/worker/src/workflows/checkout.workflow.ts

## Dependencies
ADR-002 policies; ADR-007
