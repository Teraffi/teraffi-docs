# Task: User Registration Service (4-5 hours)
**ADR:** ADR-020  
**Date:** 2025-10-01

## Inputs
- User schema
- Email service
- Tenant context

## Outputs
- Registration service
- Email verification
- Default role assignment

## Acceptance Criteria
- [ ] Register new user with email/password
- [ ] Validate email uniqueness
- [ ] Hash password
- [ ] Assign to tenant
- [ ] Assign default role
- [ ] Send verification email
- [ ] Log registration event
- [ ] Unit tests

## Implementation Details
Create user registration endpoint. Validate input, hash password, create user record. Assign default role based on tenant. Send email verification link. Log audit event.

## Files to Create
- packages/auth-service/src/registration.ts
- packages/auth-service/test/registration.spec.ts

## Dependencies
Tasks 020-001, 020-002, 020-003