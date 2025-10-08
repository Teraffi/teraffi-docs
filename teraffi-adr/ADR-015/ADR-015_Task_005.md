# Task: Goal Alignment Scoring Component (5-7 hours)
**ADR:** ADR-015  
**Date:** 2025-10-01

## Inputs
- User goals (time-weighted)
- Partner entity data from Neo4j
- LLM for qualitative evaluation

## Outputs
- Goal alignment scorer
- Demographic/geographic overlap calculation
- LLM-based capability fit assessment

## Acceptance Criteria
- [ ] Calculate demographic overlap
- [ ] Calculate geographic overlap
- [ ] LLM evaluation of qualitative fit
- [ ] Temporal weighting of goals
- [ ] Return score 0.0-1.0 with reasoning
- [ ] Handle missing data gracefully
- [ ] p95 latency <1 second
- [ ] Unit and integration tests

## Implementation Details
Query Neo4j for partner capabilities, demographics, geographies. Calculate overlap with user goals. Use LLM for qualitative assessment of capability fit. Apply temporal weighting to recent goals.

## Files to Create
- packages/affinity-engine/src/components/goal-alignment.ts
- packages/affinity-engine/test/goal-alignment.spec.ts

## Dependencies
Task 015-003, Task 015-004, ADR-011 (Neo4j), ADR-014 (LLM Gateway)