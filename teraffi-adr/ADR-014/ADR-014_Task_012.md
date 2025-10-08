# Task: REST API Endpoints (3-4 hours)
**ADR:** ADR-014  
**Date:** 2025-10-01

## Inputs
- LLM Gateway service
- Fastify API server
- Authentication middleware

## Outputs
- REST endpoints for completions and embeddings
- Admin endpoints
- OpenAPI spec

## Acceptance Criteria
- [ ] POST /api/v1/llm/completions
- [ ] POST /api/v1/llm/embeddings
- [ ] GET /api/v1/llm/costs (admin)
- [ ] DELETE /api/v1/llm/cache (admin)
- [ ] Authentication and tenant isolation
- [ ] Rate limiting
- [ ] OpenAPI spec updated
- [ ] API tests

## Implementation Details
Expose LLM Gateway through REST endpoints. Apply authentication, tenant filtering, rate limiting. Provide admin endpoints for cost monitoring and cache management.

## Files to Create
- packages/api/src/routes/llm.ts
- packages/api/src/routes/admin/llm.ts
- packages/api/test/llm.e2e.spec.ts
- docs/api/openapi-llm.yaml

## Dependencies
Task 014-008, ADR-001 (API)