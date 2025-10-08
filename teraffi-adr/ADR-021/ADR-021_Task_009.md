# Task: Result Fusion & Deduplication (4-5 hours)
**ADR:** ADR-021  
**Date:** 2025-10-01

## Inputs
- Results from multiple search sources
- Source weights

## Outputs
- Fusion algorithm
- Deduplicated results
- Combined scoring

## Acceptance Criteria
- [ ] Implement Reciprocal Rank Fusion (RRF)
- [ ] Combine text, semantic, and graph results
- [ ] Apply source weights (text: 0.4, semantic: 0.3, graph: 0.3)
- [ ] Deduplicate by entity ID
- [ ] Preserve best metadata per entity
- [ ] Sort by fused score
- [ ] Unit tests with fixtures

## Implementation Details
Use RRF algorithm to combine results from multiple sources. Weight sources appropriately. Deduplicate while preserving best result data. Sort by combined score.

## Files to Create
- packages/search/src/fusion.ts
- packages/search/src/algorithms/rrf.ts
- packages/search/test/fusion.spec.ts

## Dependencies
Tasks 021-005, 021-006, 021-007