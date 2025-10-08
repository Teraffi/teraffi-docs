# Task: Personalization Layer (3-4 hours)
**ADR:** ADR-021  
**Date:** 2025-10-01

## Inputs
- Search results
- User search history
- User preferences

## Outputs
- Personalization service
- History-based boosting
- Diversity optimization

## Acceptance Criteria
- [ ] Load user search history
- [ ] Boost entities similar to clicked items
- [ ] Penalize entities seen many times
- [ ] Promote diversity in results
- [ ] Configurable personalization strength
- [ ] A/B testable
- [ ] Unit tests

## Implementation Details
Track user search behavior. Boost results similar to previously clicked items. Penalize over-shown results. Balance personalization with diversity to avoid filter bubbles.

## Files to Create
- packages/search/src/personalization.ts
- packages/search/test/personalization.spec.ts

## Dependencies
Task 021-010