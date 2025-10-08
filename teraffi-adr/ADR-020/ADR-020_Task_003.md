# Task: JWT Token Generation & Verification (3-4 hours)
**ADR:** ADR-020  
**Date:** 2025-10-01

## Inputs
- jsonwebtoken library
- Token payload requirements

## Outputs
- JWT service
- Token generation
- Token verification
- Claims extraction

## Acceptance Criteria
- [ ] Generate JWT with user, tenant, roles, permissions
- [ ] Sign with RS256 or HS256
- [ ] Set expiry (1 hour)
- [ ] Verify token signature
- [ ] Extract claims from token
- [ ] Handle expired tokens
- [ ] Refresh token generation
- [ ] Unit tests

## Implementation Details
Use jsonwebtoken library. Generate tokens with user identity and permissions. Set appropriate expiry. Implement verification middleware. Store JWT secret securely.

## Files to Create
- packages/auth-service/src/jwt-service.ts
- packages/auth-service/src/types/jwt-payload.ts
- packages/auth-service/test/jwt-service.spec.ts

## Dependencies
Task 020-002