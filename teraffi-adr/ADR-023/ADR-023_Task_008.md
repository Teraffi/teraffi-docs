# Task: Payment Method Management (4-5 hours)
**ADR:** ADR-023  
**Date:** 2025-10-01

## Inputs
- Stripe payment methods
- PCI compliance requirements

## Outputs
- Payment method service
- Card management
- Payment method UI components

## Acceptance Criteria
- [ ] Add payment method
- [ ] Update default payment method
- [ ] Remove payment method
- [ ] Display masked card info (last 4 digits)
- [ ] Handle 3D Secure authentication
- [ ] Never store full card numbers
- [ ] PCI DSS compliant
- [ ] Unit tests

## Implementation Details
Use Stripe Payment Methods API. Never handle raw card data. Support adding, updating, removing payment methods. Display masked card information. Handle SCA/3DS authentication flows.

## Files to Create
- packages/billing/src/payment-method-service.ts
- packages/billing/test/payment-method-service.spec.ts

## Dependencies
Task 023-004