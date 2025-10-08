# Task: Comprehensive Test Suite (5-6 hours)
**ADR:** ADR-020  
**Date:** 2025-10-01

## Inputs
- All auth components
- Security test scenarios

## Outputs
- Unit tests
- Integration tests
- Security tests
- E2E flows

## Acceptance Criteria
- [ ] Unit tests for all services
- [ ] Integration tests for auth flows
- [ ] Test password hashing
- [ ] Test JWT generation/verification
- [ ] Test MFA flows
- [ ] Test SSO flows
- [ ] Test permission checks
- [ ] Test account lockout
- [ ] Test rate limiting
- [ ] Security penetration tests
- [ ] CI green

## Implementation Details
Create comprehensive test suite. Test all authentication and authorization paths. Include security tests (SQL injection, XSS, brute force). Test edge cases and error conditions.

## Files to Create
- packages/auth-service/test/integration.spec.ts
- packages/auth-service/test/security.spec.ts
- packages/api/test/auth-flows.e2e.spec.ts

## Dependencies
Tasks 020-001 through 020-016