# Task: Billing Analytics Dashboard (5-6 hours)
**ADR:** ADR-023  
**Date:** 2025-10-01

## Inputs
- Subscription data
- Revenue data
- Churn data

## Outputs
- Analytics service
- Metrics calculation
- Admin dashboard

## Acceptance Criteria
- [ ] Calculate MRR (Monthly Recurring Revenue)
- [ ] Calculate ARR (Annual Recurring Revenue)
- [ ] Calculate churn rate
- [ ] Calculate LTV (Lifetime Value)
- [ ] Track new subscriptions
- [ ] Track upgrades/downgrades
- [ ] Track cancellations
- [ ] Revenue cohort analysis
- [ ] Admin dashboard
- [ ] Unit tests

## Implementation Details
Build analytics tracking key SaaS metrics. Calculate MRR, ARR, churn, LTV. Track subscription movements. Create dashboard for finance team.

## Files to Create
- packages/billing/src/analytics.ts
- apps/admin/src/pages/billing/analytics.tsx
- packages/billing/test/analytics.spec.ts

## Dependencies
Tasks 023-001, 023-007