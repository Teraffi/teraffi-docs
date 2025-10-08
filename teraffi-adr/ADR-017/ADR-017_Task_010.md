# Task: Neo4j Feedback Loop Integration (3-4 hours)
**ADR:** ADR-017  
**Date:** 2025-10-01

## Inputs
- Deal outcomes
- Success scores
- Partnership data

## Outputs
- Neo4j updater
- PARTNERS_WITH relationship updates
- Success metrics storage

## Acceptance Criteria
- [ ] Update PARTNERS_WITH on signed/active/completed
- [ ] Store deal_value, start_date, end_date
- [ ] Update success_score on completion
- [ ] Track partnership status
- [ ] Feed data to ADR-015 (Historical Performance)
- [ ] Integration tests
- [ ] Idempotent updates

## Implementation Details
When deals reach key stages, update Neo4j PARTNERS_WITH relationships. Store deal metadata. On completion, calculate and store success score for use by Affinity Engine's historical performance component.

## Files to Create
- packages/deal-flow/src/neo4j-updater.ts
- packages/deal-flow/test/neo4j-integration.spec.ts

## Dependencies
Task 017-002, Task 017-007, ADR-011 (Neo4j)