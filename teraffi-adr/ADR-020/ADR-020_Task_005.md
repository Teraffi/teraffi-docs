# Task: Login & Authentication Flow (5-6 hours)
**ADR:** ADR-020  
**Date:** 2025-10-01

## Inputs
- User credentials
- JWT service
- Password utilities

## Outputs
- Login service
- Token issuance
- Account lockout logic
- Audit logging

## Acceptance Criteria
- [ ] Authenticate with email/password
- [ ] Verify password
- [ ] Check account active status
- [ ] Handle account lockout (5 failed attempts = 30 min lock)
- [ ] Load user permissions
- [ ] Generate access and refresh tokens
- [ ] Update last_login_at
- [ ] Log authentication events
- [ ] Unit tests

## Implementation Details
Implement login endpoint. Verify credentials, check account status, handle lockouts. Load roles/permissions. Generate JWT tokens. Log all authentication attempts.

## Files to Create
- packages/auth-service/src/auth-service.ts
- packages/auth-service/src/lockout-handler.ts
- packages/auth-service/test/auth-service.spec.ts

## Dependencies
Tasks 020-002, 020-003