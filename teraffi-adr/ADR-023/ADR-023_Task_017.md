# Task: Revenue Recognition Tracking (4-5 hours)
**ADR:** ADR-023  
**Date:** 2025-10-01

## Inputs
- Subscription data
- Accounting requirements (ASC 606)

## Outputs
- Revenue recognition service
- Deferred revenue calculation
- Accounting reports

## Acceptance Criteria
- [ ] Calculate recognized revenue per period
- [ ] Track deferred revenue
- [ ] Handle proration correctly
- [ ] Support refunds
- [ ] Monthly revenue reports
- [ ] Integration with accounting system (future)
- [ ] Audit trail
- [ ] Unit tests

## Implementation Details
Track revenue recognition following ASC 606. Calculate recognized vs deferred revenue. Handle proration, refunds, cancellations. Generate reports for accounting.

## Files to Create
- packages/billing/src/revenue-recognition.ts
- packages/billing/src/reports/revenue-report.ts
- packages/billing/test/revenue-recognition.spec.ts

## Dependencies
Task 023-007