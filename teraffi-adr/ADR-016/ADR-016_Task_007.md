# Task: Momentum & Velocity Calculator (6-8 hours)
**ADR:** ADR-016  
**Date:** 2025-10-01

## Inputs
- Time-series signal data
- Statistical analysis algorithms

## Outputs
- Momentum calculator
- Velocity and acceleration metrics
- Lifecycle stage detection
- Peak prediction

## Acceptance Criteria
- [ ] Calculate momentum score (0-1) from signal counts
- [ ] Calculate velocity (rate of change)
- [ ] Calculate acceleration (rate of velocity change)
- [ ] Determine lifecycle stage (emerging/growing/peak/declining)
- [ ] Predict peak date for growing trends
- [ ] Require minimum 7 days of data
- [ ] Unit tests with time-series fixtures

## Implementation Details
Aggregate daily signal counts. Calculate moving averages. Compute velocity as rate of change between weeks. Determine lifecycle from momentum + velocity patterns. Linear projection for peak prediction.

## Files to Create
- packages/social-listening/src/momentum-calculator.ts
- packages/social-listening/src/analytics/time-series-analysis.ts
- packages/social-listening/test/momentum-calculator.spec.ts

## Dependencies
Task 016-006