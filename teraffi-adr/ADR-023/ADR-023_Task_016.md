# Task: Tax Calculation (Stripe Tax) (3-4 hours)
**ADR:** ADR-023  
**Date:** 2025-10-01

## Inputs
- Stripe Tax
- Customer location

## Outputs
- Tax calculation
- Tax reporting

## Acceptance Criteria
- [ ] Enable Stripe Tax
- [ ] Collect customer location
- [ ] Calculate tax automatically
- [ ] Display tax on invoices
- [ ] Handle tax exemptions
- [ ] Support reverse charge (EU VAT)
- [ ] Tax reporting for compliance
- [ ] Unit tests

## Implementation Details
Use Stripe Tax for automatic tax calculation. Collect customer location during checkout. Apply appropriate tax rates. Handle exemptions. Support international tax rules.

## Files to Create
- packages/billing/src/tax-service.ts
- packages/billing/test/tax-service.spec.ts

## Dependencies
Task 023-004