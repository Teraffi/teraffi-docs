# Task: Core Billing Service (6-8 hours)
**ADR:** ADR-023  
**Date:** 2025-10-01

## Inputs
- Stripe client
- Plan configuration
- Database schema

## Outputs
- Billing service
- Subscription CRUD
- Customer management

## Acceptance Criteria
- [ ] Create Stripe customer
- [ ] Create subscription
- [ ] Update subscription (upgrade/downgrade)
- [ ] Cancel subscription (immediate or end of period)
- [ ] Get subscription details
- [ ] Attach payment method
- [ ] Handle proration calculations
- [ ] Sync with database
- [ ] Unit tests

## Implementation Details
Build core billing service wrapping Stripe API. Handle customer creation, subscription lifecycle, payment methods. Sync all changes to local database. Support proration on plan changes.

## Files to Create
- packages/billing/src/billing-service.ts
- packages/billing/src/customer-manager.ts
- packages/billing/test/billing-service.spec.ts

## Dependencies
Tasks 023-001, 023-002, 023-003