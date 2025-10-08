# Task: Cultural Trend Momentum Scoring (5-7 hours)
**ADR:** ADR-015  
**Date:** 2025-10-01

## Inputs
- Shared trends between entities (Neo4j)
- Trend lifecycle data
- Social listening signals (optional)

## Outputs
- Cultural momentum scorer
- Trend lifecycle analysis
- Predictive timing scores

## Acceptance Criteria
- [ ] Find shared trends between entities
- [ ] Score based on trend lifecycle position
- [ ] Bonus for emerging trends (0-90 days)
- [ ] Penalty for post-peak trends
- [ ] Optional real-time social velocity integration
- [ ] Return score 0.0-1.5 (cap for exceptional trends)
- [ ] p95 latency <1 second
- [ ] Unit tests with trend examples

## Implementation Details
Query Neo4j for shared trends. Analyze emergence date vs current date to determine lifecycle position. Apply multipliers based on "skate to where puck is going" philosophy (emerging trends score higher).

## Files to Create
- packages/affinity-engine/src/components/cultural-momentum.ts
- packages/affinity-engine/src/social-listening.ts (stub)
- packages/affinity-engine/test/cultural-momentum.spec.ts

## Dependencies
Task 015-003, ADR-011 (Neo4j)