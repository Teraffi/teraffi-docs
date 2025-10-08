# Task: Usage Tracking & Metering (5-6 hours)
**ADR:** ADR-023  
**Date:** 2025-10-01

## Inputs
- Usage events
- Metered billing requirements
- Stripe metering API

## Outputs
- Usage tracker
- Stripe reporting
- Quota enforcement

## Acceptance Criteria
- [ ] Track usage events (partnerships, API calls, etc.)
- [ ] Store usage records in database
- [ ] Report usage to Stripe for metered billing
- [ ] Aggregate usage by billing period
- [ ] Check quota limits
- [ ] Alert on quota approaching
- [ ] Support usage-based add-ons
- [ ] Unit tests

## Implementation Details
Track usage events. Store in database. Report to Stripe for metered billing. Check against plan quotas. Alert when approaching limits. Support overage billing.

## Files to Create
- packages/billing/src/usage-tracker.ts
- packages/billing/src/quota-checker.ts
- packages/billing/test/usage-tracker.spec.ts

## Dependencies
Tasks 023-001, 023-004