# Task: Checkout Flow (5-6 hours)
**ADR:** ADR-023  
**Date:** 2025-10-01

## Inputs
- Stripe Checkout
- Plan selection

## Outputs
- Checkout session creation
- Success/cancel handling
- Trial support

## Acceptance Criteria
- [ ] Create Stripe Checkout session
- [ ] Support plan selection
- [ ] Support trial periods
- [ ] Handle success redirect
- [ ] Handle cancel redirect
- [ ] Confirm subscription after checkout
- [ ] Send confirmation email
- [ ] Handle setup intents (no immediate charge)
- [ ] Integration tests

## Implementation Details
Use Stripe Checkout for payment collection. Create checkout session with selected plan. Support trial periods. Handle success/cancel redirects. Confirm subscription creation.

## Files to Create
- packages/billing/src/checkout-service.ts
- packages/api/src/routes/checkout.ts
- packages/billing/test/checkout-service.spec.ts

## Dependencies
Task 023-004