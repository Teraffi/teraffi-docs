# Task: Password Hashing & Validation (2-3 hours)
**ADR:** ADR-020  
**Date:** 2025-10-01

## Inputs
- bcrypt library
- Password policy requirements

## Outputs
- Password utilities
- Strength validation
- Hash generation

## Acceptance Criteria
- [ ] Hash passwords with bcrypt (cost factor 12)
- [ ] Validate password strength (min 12 chars, uppercase, lowercase, number, special)
- [ ] Compare passwords securely
- [ ] Prevent timing attacks
- [ ] Common password blacklist (optional)
- [ ] Unit tests
- [ ] Performance <200ms for hash

## Implementation Details
Implement secure password hashing using bcrypt. Validate password strength requirements. Use constant-time comparison. Consider async hashing to prevent blocking.

## Files to Create
- packages/auth-service/src/utils/password.ts
- packages/auth-service/test/password.spec.ts

## Dependencies
None