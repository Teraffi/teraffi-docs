# Task: Historical Deal Performance Scoring (5-6 hours)
**ADR:** ADR-015  
**Date:** 2025-10-01

## Inputs
- Partnership outcomes from PostgreSQL
- Affinity pattern data
- Deal success scores

## Outputs
- Historical performance scorer
- Similar deal identification
- Recency-weighted success rates

## Acceptance Criteria
- [ ] Find similar past deals (entities or affinity patterns)
- [ ] Calculate weighted success rate by deal value
- [ ] Apply recency weighting (exponential decay)
- [ ] Require 50%+ affinity overlap for similarity
- [ ] Return score 0.0-1.0
- [ ] Handle zero history (neutral 0.5 score)
- [ ] Unit tests with deal fixtures

## Implementation Details
Query partnership_outcomes table for similar deals. Filter by affinity pattern overlap. Weight success by deal value and recency. Build confidence based on sample size.

## Files to Create
- packages/affinity-engine/src/components/historical-performance.ts
- db/migrations/XXX_create_partnership_outcomes.sql
- packages/affinity-engine/test/historical-performance.spec.ts

## Dependencies
Task 015-003, ADR-007 (PostgreSQL)