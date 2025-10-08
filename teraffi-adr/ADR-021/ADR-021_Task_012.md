# Task: Main Search Service Orchestration (5-6 hours)
**ADR:** ADR-021  
**Date:** 2025-10-01

## Inputs
- All search components
- Search request schema

## Outputs
- Main search service
- Multi-stage pipeline
- Result formatting

## Acceptance Criteria
- [ ] Orchestrate: parse → search → fuse → rerank → personalize
- [ ] Execute searches in parallel where possible
- [ ] Handle errors gracefully (fallback strategies)
- [ ] Format results consistently
- [ ] Extract and return facets
- [ ] Pagination
- [ ] Total count
- [ ] Performance <200ms (p95)
- [ ] Integration tests

## Implementation Details
Build main search service that orchestrates entire pipeline. Parse query, execute parallel searches, fuse results, rerank with affinity, personalize. Return formatted results.

## Files to Create
- packages/search/src/search-service.ts
- packages/search/test/search-service.spec.ts
- packages/search/test/integration.spec.ts

## Dependencies
Tasks 021-005 through 021-011