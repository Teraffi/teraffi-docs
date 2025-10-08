# Task: Faceted Search & Aggregations (3-4 hours)
**ADR:** ADR-021  
**Date:** 2025-10-01

## Inputs
- Search results
- Facet definitions

## Outputs
- Facet extraction
- Aggregation queries
- Filter counts

## Acceptance Criteria
- [ ] Return facets: industries, values, demographics, locations
- [ ] Show document counts per facet value
- [ ] Multi-select facets (OR logic within facet, AND across facets)
- [ ] Range facets (affinity_score, date ranges)
- [ ] Exclude selected filters from facet counts
- [ ] Performance impact minimal
- [ ] Unit tests

## Implementation Details
Use Elasticsearch aggregations to extract facets. Return counts for each facet value. Support multi-select filters. Handle range facets for numeric fields.

## Files to Create
- packages/search/src/facets.ts
- packages/search/test/facets.spec.ts

## Dependencies
Task 021-005