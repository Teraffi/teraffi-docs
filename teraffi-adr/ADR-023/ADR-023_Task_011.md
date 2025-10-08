# Task: Dunning & Failed Payment Recovery (5-6 hours)
**ADR:** ADR-023  
**Date:** 2025-10-01

## Inputs
- Failed payment events
- Retry strategies

## Outputs
- Dunning service
- Retry scheduler
- Customer notifications

## Acceptance Criteria
- [ ] Track payment attempts
- [ ] Automatic retry logic (Stripe Smart Retries)
- [ ] Send notifications on payment failure
- [ ] Escalating email sequence (day 1, 3, 7)
- [ ] Grace period before suspension
- [ ] Suspend account after repeated failures
- [ ] Reactivate on successful payment
- [ ] Recovery rate tracking
- [ ] Unit tests

## Implementation Details
Implement dunning to recover failed payments. Track payment attempts. Send escalating notifications. Provide grace period. Suspend account after multiple failures. Reactivate automatically on payment success.

## Files to Create
- packages/billing/src/dunning-service.ts
- packages/billing/src/jobs/process-failed-payments.ts
- packages/billing/test/dunning-service.spec.ts

## Dependencies
Tasks 023-006, 023-007, ADR-022 (Notifications)