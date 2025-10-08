# Task: Autocomplete Service (4-5 hours)
**ADR:** ADR-021  
**Date:** 2025-10-01

## Inputs
- Elasticsearch ngram analyzer
- Entity names
- Trending searches

## Outputs
- Autocomplete service
- Entity suggestions
- Trend suggestions
- Popular queries

## Acceptance Criteria
- [ ] Prefix matching on entity names
- [ ] Ngram matching for partial words
- [ ] Include trending searches
- [ ] Return entity type (brand, member, content, trend)
- [ ] Include entity ID for direct navigation
- [ ] Limit results (default 10)
- [ ] Performance <50ms
- [ ] Unit tests

## Implementation Details
Use Elasticsearch prefix and ngram queries for autocomplete. Combine entity suggestions with trending queries. Return quickly for responsive UI.

## Files to Create
- packages/search/src/autocomplete.ts
- packages/search/test/autocomplete.spec.ts

## Dependencies
Task 021-002