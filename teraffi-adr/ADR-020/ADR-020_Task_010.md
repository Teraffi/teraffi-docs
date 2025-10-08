# Task: Password Reset Flow (4-5 hours)
**ADR:** ADR-020  
**Date:** 2025-10-01

## Inputs
- Email service
- Token generation

## Outputs
- Password reset service
- Reset tokens
- Email templates

## Acceptance Criteria
- [ ] Request password reset (email)
- [ ] Generate secure reset token
- [ ] Send reset email with link
- [ ] Validate reset token (not expired, not used)
- [ ] Reset password with token
- [ ] Invalidate token after use
- [ ] Token expiry (1 hour)
- [ ] Log password changes
- [ ] Unit tests

## Implementation Details
Implement password reset flow. Generate secure tokens with expiry. Send email with reset link. Validate token before allowing password change. Invalidate token after single use.

## Files to Create
- packages/auth-service/src/password-reset.ts
- packages/auth-service/src/templates/password-reset-email.ts
- packages/auth-service/test/password-reset.spec.ts

## Dependencies
Tasks 020-002, 020-005