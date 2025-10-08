# Task: Stripe Webhook Handler (6-8 hours)
**ADR:** ADR-023  
**Date:** 2025-10-01

## Inputs
- Stripe webhook events
- Event types
- Signature verification

## Outputs
- Webhook receiver
- Event router
- Event handlers

## Acceptance Criteria
- [ ] Receive and verify webhook signatures
- [ ] Route events to appropriate handlers
- [ ] Handle subscription.created
- [ ] Handle subscription.updated
- [ ] Handle subscription.deleted
- [ ] Handle invoice.paid
- [ ] Handle invoice.payment_failed
- [ ] Handle payment_intent events
- [ ] Log all events to billing_events table
- [ ] Idempotent processing
- [ ] Integration tests

## Implementation Details
Build webhook endpoint receiving Stripe events. Verify signatures. Route to event-specific handlers. Update database based on events. Ensure idempotent processing.

## Files to Create
- packages/billing/src/webhook-handler.ts
- packages/billing/src/handlers/subscription-handlers.ts
- packages/billing/src/handlers/invoice-handlers.ts
- packages/billing/src/handlers/payment-handlers.ts
- packages/billing/test/webhook-handler.spec.ts

## Dependencies
Task 023-004