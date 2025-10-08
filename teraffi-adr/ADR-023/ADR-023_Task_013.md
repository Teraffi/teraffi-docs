# Task: Billing Dashboard UI (6-8 hours)
**ADR:** ADR-023  
**Date:** 2025-10-01

## Inputs
- Billing API
- Design requirements

## Outputs
- Subscription management UI
- Usage dashboard
- Invoice history

## Acceptance Criteria
- [ ] Current plan display
- [ ] Upgrade/downgrade buttons
- [ ] Usage metrics with progress bars
- [ ] Invoice history table
- [ ] Payment method display
- [ ] Update payment method button → Portal
- [ ] Cancel subscription flow
- [ ] Invoice download links
- [ ] Responsive design
- [ ] E2E tests

## Implementation Details
Build billing dashboard showing current subscription, usage, invoices. Allow plan changes. Display payment method. Link to Stripe Portal for payment updates. Show usage vs quotas.

## Files to Create
- apps/web/src/pages/billing/index.tsx
- apps/web/src/pages/billing/plans.tsx
- apps/web/src/components/SubscriptionCard.tsx
- apps/web/src/components/UsageMetrics.tsx
- apps/web/src/components/InvoiceList.tsx
- apps/web/test/billing.e2e.spec.ts

## Dependencies
Task 023-012