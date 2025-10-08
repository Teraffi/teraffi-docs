# Task: JWT Middleware (Azure AD B2C) (4-6 hours)
**ADR:** ADR-004  
**Date:** 2025-10-01

## Inputs
- OIDC issuer
- JWKS URL

## Outputs
- Validator middleware

## Acceptance Criteria
- [ ] Signature verified
- [ ] exp/nbf/aud/iss enforced
- [ ] Tenant/roles extracted

## Implementation Details
- Implement JWKS cache; map claims; error mapping.

## Files to Create
- packages/api/src/middleware/jwt.ts
- packages/api/test/jwt.spec.ts

## Dependencies
ADR-001 API
