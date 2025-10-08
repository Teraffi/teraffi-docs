# Task: Plan Upgrade/Downgrade Flow (5-6 hours)
**ADR:** ADR-023  
**Date:** 2025-10-01

## Inputs
- Current subscription
- Target plan
- Proration rules

## Outputs
- Upgrade/downgrade service
- Proration calculation
- Confirmation UI

## Acceptance Criteria
- [ ] Calculate proration amount
- [ ] Preview cost changes
- [ ] Confirm upgrade/downgrade
- [ ] Handle immediate changes
- [ ] Handle end-of-period changes
- [ ] Update features immediately on upgrade
- [ ] Grace period on downgrade
- [ ] Send confirmation email
- [ ] Integration tests

## Implementation Details
Build service handling plan changes. Calculate proration. Show preview before confirming. Update features based on new plan. Handle immediate vs end-of-period changes.

## Files to Create
- packages/billing/src/plan-change-service.ts
- apps/web/src/pages/billing/change-plan.tsx
- packages/billing/test/plan-change.spec.ts

## Dependencies
Task 023-004