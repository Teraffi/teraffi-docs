# Task: Authentication REST API (4-5 hours)
**ADR:** ADR-020  
**Date:** 2025-10-01

## Inputs
- All auth services
- API specifications

## Outputs
- REST endpoints
- OpenAPI documentation
- Request validation

## Acceptance Criteria
- [ ] POST /api/v1/auth/register
- [ ] POST /api/v1/auth/login
- [ ] POST /api/v1/auth/refresh
- [ ] POST /api/v1/auth/logout
- [ ] POST /api/v1/auth/password-reset/request
- [ ] POST /api/v1/auth/password-reset/confirm
- [ ] POST /api/v1/auth/mfa/enable
- [ ] POST /api/v1/auth/mfa/verify
- [ ] GET /api/v1/auth/me
- [ ] Request validation
- [ ] OpenAPI spec
- [ ] API tests

## Implementation Details
Expose all auth services via REST API. Implement proper request validation. Return appropriate status codes. Document with OpenAPI spec.

## Files to Create
- packages/api/src/routes/auth.ts
- packages/api/test/auth.e2e.spec.ts
- docs/api/openapi-auth.yaml

## Dependencies
Tasks 020-004 through 020-012