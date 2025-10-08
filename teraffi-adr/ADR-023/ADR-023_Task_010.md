# Task: Customer Portal Integration (4-5 hours)
**ADR:** ADR-023  
**Date:** 2025-10-01

## Inputs
- Stripe Customer Portal
- Self-service requirements

## Outputs
- Portal session creation
- Portal configuration

## Acceptance Criteria
- [ ] Create Stripe Portal session
- [ ] Configure portal features (update card, cancel sub, view invoices)
- [ ] Generate portal URL
- [ ] Handle return URL
- [ ] Sync changes from portal to database
- [ ] Portal branding configuration
- [ ] Unit tests

## Implementation Details
Use Stripe Customer Portal for self-service subscription management. Configure allowed features. Generate portal URLs. Handle webhooks from portal actions.

## Files to Create
- packages/billing/src/portal-service.ts
- packages/api/src/routes/portal.ts
- packages/billing/test/portal-service.spec.ts

## Dependencies
Task 023-006