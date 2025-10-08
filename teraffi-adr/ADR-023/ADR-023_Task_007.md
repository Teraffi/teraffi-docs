# Task: Invoice Management (4-5 hours)
**ADR:** ADR-023  
**Date:** 2025-10-01

## Inputs
- Stripe invoice data
- Tax requirements

## Outputs
- Invoice service
- Invoice storage
- PDF generation

## Acceptance Criteria
- [ ] Sync invoices from Stripe
- [ ] Store invoice details in database
- [ ] Get invoice list for tenant
- [ ] Get invoice PDF
- [ ] Track payment status
- [ ] Calculate tax (Stripe Tax)
- [ ] Display invoice details
- [ ] Unit tests

## Implementation Details
Sync invoice data from Stripe webhooks. Store in database. Provide API to retrieve invoices. Link to Stripe-hosted PDF. Track payment status and history.

## Files to Create
- packages/billing/src/invoice-service.ts
- packages/billing/test/invoice-service.spec.ts

## Dependencies
Task 023-006