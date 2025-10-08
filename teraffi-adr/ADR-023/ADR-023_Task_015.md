# Task: Trial Period Management (4-5 hours)
**ADR:** ADR-023  
**Date:** 2025-10-01

## Inputs
- Trial configuration
- Subscription events

## Outputs
- Trial service
- Trial expiration handling
- Trial notifications

## Acceptance Criteria
- [ ] Start trial on signup
- [ ] Track trial end date
- [ ] Notify 7 days before trial ends
- [ ] Notify 3 days before trial ends
- [ ] Notify on trial end day
- [ ] Require payment method before trial ends
- [ ] Convert to paid on trial end
- [ ] Handle trial cancellation
- [ ] Unit tests

## Implementation Details
Implement trial period management. Set trial duration on subscription creation. Send reminder notifications. Require payment method. Convert to paid subscription automatically or cancel.

## Files to Create
- packages/billing/src/trial-service.ts
- packages/billing/src/jobs/process-trial-endings.ts
- packages/billing/test/trial-service.spec.ts

## Dependencies
Tasks 023-004, ADR-022 (Notifications)