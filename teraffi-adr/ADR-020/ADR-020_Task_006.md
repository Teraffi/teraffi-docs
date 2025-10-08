# Task: Refresh Token Management (3-4 hours)
**ADR:** ADR-020  
**Date:** 2025-10-01

## Inputs
- Refresh token schema
- Redis for token blacklist (optional)

## Outputs
- Refresh token service
- Token rotation
- Revocation support

## Acceptance Criteria
- [ ] Generate refresh tokens
- [ ] Store hashed tokens in database
- [ ] Validate refresh tokens
- [ ] Issue new access tokens
- [ ] Rotate refresh tokens on use (optional)
- [ ] Revoke tokens on logout
- [ ] Clean up expired tokens
- [ ] Unit tests

## Implementation Details
Implement refresh token generation and storage. Hash tokens before storage. Validate and issue new access tokens. Support token revocation. Scheduled job to clean expired tokens.

## Files to Create
- packages/auth-service/src/refresh-token-service.ts
- packages/auth-service/src/jobs/cleanup-tokens.ts
- packages/auth-service/test/refresh-token.spec.ts

## Dependencies
Task 020-005