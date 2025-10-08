# Task: Affinity-Based Reranking (4-5 hours)
**ADR:** ADR-021  
**Date:** 2025-10-01

## Inputs
- Fused search results (top 100)
- Affinity scores from database
- User ID

## Outputs
- Reranking service
- Affinity score integration
- Boosted relevance

## Acceptance Criteria
- [ ] Load affinity scores for user → candidates
- [ ] Boost search score with affinity score
- [ ] Weighted combination (search: 0.6, affinity: 0.4)
- [ ] Re-sort by boosted score
- [ ] Handle missing affinity scores (default 0)
- [ ] Performance <100ms for 100 entities
- [ ] Unit tests

## Implementation Details
After initial search, load affinity scores for top candidates. Combine search relevance with affinity for personalized ranking. Boost entities with high affinity to user.

## Files to Create
- packages/search/src/reranking.ts
- packages/search/test/reranking.spec.ts

## Dependencies
Task 021-009, ADR-015 (Affinity Scoring)