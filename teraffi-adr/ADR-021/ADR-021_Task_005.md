# Task: Full-Text Search Implementation (5-6 hours)
**ADR:** ADR-021  
**Date:** 2025-10-01

## Inputs
- Elasticsearch index
- Query requirements

## Outputs
- Full-text search service
- Query builder
- Faceted filtering

## Acceptance Criteria
- [ ] Multi-match queries across name, description, bio
- [ ] Fuzzy matching (AUTO fuzziness)
- [ ] Field boosting (name^3, description^2)
- [ ] Tenant isolation filter
- [ ] Entity type filtering
- [ ] Industry, location, values facets
- [ ] Range filters (affinity_score, dates)
- [ ] Highlighting
- [ ] Pagination
- [ ] Unit tests

## Implementation Details
Build Elasticsearch query DSL for full-text search. Support multi-match with field boosting. Apply filters for facets. Implement highlighting. Return results with metadata.

## Files to Create
- packages/search/src/full-text-search.ts
- packages/search/src/query-builder.ts
- packages/search/test/full-text-search.spec.ts

## Dependencies
Task 021-004