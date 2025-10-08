# Task: Workflow Wiring into API (start, signal, query) (4-6 hours)
**ADR:** ADR-008  
**Date:** 2025-10-01

## Inputs
- API routes
- Temporal client

## Outputs
- Start/signal/query endpoints

## Acceptance Criteria
- [ ] Start returns 202
- [ ] Signal advances state
- [ ] Query returns snapshot

## Implementation Details
- Add routes under /v1/checkout and /v1/fulfillment.

## Files to Create
- packages/api/src/routes/checkout.ts
- packages/api/src/routes/fulfillment.ts

## Dependencies
ADR-001/002
