# Task: Search REST API (4-5 hours)
**ADR:** ADR-021  
**Date:** 2025-10-01

## Inputs
- Search service
- Autocomplete service
- API specifications

## Outputs
- REST endpoints
- Request validation
- OpenAPI documentation

## Acceptance Criteria
- [ ] GET /api/v1/search - main search endpoint
- [ ] GET /api/v1/search/autocomplete - suggestions
- [ ] GET /api/v1/search/popular - popular queries
- [ ] Request validation (query params)
- [ ] Response formatting
- [ ] Error handling
- [ ] Rate limiting
- [ ] OpenAPI spec
- [ ] API tests

## Implementation Details
Expose search services via REST API. Validate request parameters. Apply rate limiting. Return consistent response format with results, facets, pagination metadata.

## Files to Create
- packages/api/src/routes/search.ts
- packages/api/test/search.e2e.spec.ts
- docs/api/openapi-search.yaml

## Dependencies
Tasks 021-012, 021-013