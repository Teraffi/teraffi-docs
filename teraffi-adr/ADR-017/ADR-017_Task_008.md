# Task: Deal Dashboard & REST API (5-6 hours)
**ADR:** ADR-017  
**Date:** 2025-10-01

## Inputs
- Deal data aggregation
- Fastify API server
- Authentication

## Outputs
- REST endpoints
- Dashboard queries
- OpenAPI spec

## Acceptance Criteria
- [ ] GET /api/v1/deals - list deals with filters
- [ ] GET /api/v1/deals/:id - deal details
- [ ] POST /api/v1/deals/:id/transition - change stage
- [ ] POST /api/v1/deals/:id/comments - add comment
- [ ] POST /api/v1/deals/:id/documents - upload document
- [ ] GET /api/v1/deals/:id/activity - activity feed
- [ ] POST /api/v1/deals/:id/milestones - create milestone
- [ ] Authentication and authorization
- [ ] OpenAPI documentation
- [ ] API tests

## Implementation Details
Expose deal flow services through REST endpoints. Implement proper access control (only participants can view/modify deals). Aggregate data for dashboard views.

## Files to Create
- packages/api/src/routes/deals.ts
- packages/api/test/deals.e2e.spec.ts
- docs/api/openapi-deals.yaml

## Dependencies
Tasks 017-002 through 017-007, ADR-001 (API)