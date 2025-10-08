# Task: Analytics REST API (5-6 hours)
**ADR:** ADR-018  
**Date:** 2025-10-01

## Inputs
- ClickHouse connection
- API requirements
- Authentication

## Outputs
- Analytics API endpoints
- Query parameterization
- Response caching

## Acceptance Criteria
- [ ] GET /api/v1/metrics/partnership-funnel
- [ ] GET /api/v1/metrics/affinity-trends
- [ ] GET /api/v1/metrics/trend-performance
- [ ] GET /api/v1/metrics/entity-summary
- [ ] Query parameter validation
- [ ] Response caching (Redis)
- [ ] Rate limiting
- [ ] OpenAPI documentation
- [ ] API tests

## Implementation Details
Build REST API exposing analytical queries. Allow parameterization for filtering by date, entity, etc. Cache responses for performance. Apply rate limiting to prevent abuse.

## Files to Create
- packages/analytics-api/src/routes/metrics.ts
- packages/analytics-api/src/queries/partnership-metrics.ts
- packages/analytics-api/test/metrics.e2e.spec.ts
- docs/api/openapi-analytics.yaml

## Dependencies
Task 018-001, Task 018-005, ADR-001 (API)