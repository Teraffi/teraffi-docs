# Task: GraphRAG REST API Endpoints (3-4 hours)
**ADR:** ADR-013  
**Date:** 2025-10-01

## Inputs
- GraphRAG service
- Fastify API server
- Authentication middleware

## Outputs
- REST endpoints for queries
- Conversation endpoints
- Admin endpoints

## Acceptance Criteria
- [ ] POST /api/v1/graphrag/query - single query
- [ ] POST /api/v1/graphrag/conversations - create conversation
- [ ] POST /api/v1/graphrag/conversations/:id/query - follow-up
- [ ] GET /api/v1/graphrag/conversations/:id - get history
- [ ] Authentication and tenant isolation
- [ ] OpenAPI spec updated
- [ ] API tests

## Implementation Details
Create REST endpoints wrapping GraphRAG service. Handle authentication, tenant context, rate limiting. Return structured responses with answers and citations.

## Files to Create
- packages/api/src/routes/graphrag.ts
- packages/api/test/graphrag.e2e.spec.ts
- docs/api/openapi-graphrag.yaml

## Dependencies
Task 013-006, ADR-001 (API), ADR-006 (Tenant filtering)