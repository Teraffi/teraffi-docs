# Task: Milestone & KPI Tracking (4-5 hours)
**ADR:** ADR-017  
**Date:** 2025-10-01

## Inputs
- Milestone definitions
- KPI updates
- Target dates

## Outputs
- Performance tracker service
- Milestone CRUD operations
- Success score calculation

## Acceptance Criteria
- [ ] Create milestones with KPIs
- [ ] Update KPI actual values
- [ ] Auto-complete when all KPIs met
- [ ] Calculate overall success score (0-5)
- [ ] Notify on milestone completion
- [ ] Track completion rates
- [ ] Dashboard-ready metrics
- [ ] Unit tests

## Implementation Details
Store milestones with associated KPIs (target/actual). Check completion when actuals updated. Calculate success score from completion + KPI achievement rates. Notify stakeholders on completions.

## Files to Create
- packages/deal-flow/src/performance-tracker.ts
- packages/deal-flow/src/calculators/success-score.ts
- packages/deal-flow/test/performance-tracker.spec.ts

## Dependencies
Task 017-001