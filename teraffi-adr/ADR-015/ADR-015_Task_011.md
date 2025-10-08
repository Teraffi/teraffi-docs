# Task: REST API Endpoints (3-4 hours)
**ADR:** ADR-015  
**Date:** 2025-10-01

## Inputs
- Affinity Engine service
- Fastify API server
- Authentication middleware

## Outputs
- REST endpoints for affinity scoring
- Batch scoring endpoint
- Explanation endpoint

## Acceptance Criteria
- [ ] POST /api/v1/affinity/score - single score
- [ ] POST /api/v1/affinity/batch - batch scoring
- [ ] GET /api/v1/affinity/:id/explain - detailed explanation
- [ ] Authentication and tenant isolation
- [ ] Rate limiting
- [ ] OpenAPI spec updated
- [ ] API tests

## Implementation Details
Expose Affinity Engine through REST endpoints. Support single and batch scoring. Provide detailed explanations. Apply authentication and rate limiting.

## Files to Create
- packages/api/src/routes/affinity.ts
- packages/api/test/affinity.e2e.spec.ts
- docs/api/openapi-affinity.yaml

## Dependencies
Task 015-009, ADR-001 (API)