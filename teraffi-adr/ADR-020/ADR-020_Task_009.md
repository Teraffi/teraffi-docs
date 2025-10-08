# Task: Multi-Factor Authentication (MFA/TOTP) (6-8 hours)
**ADR:** ADR-020  
**Date:** 2025-10-01

## Inputs
- TOTP library (otplib)
- QR code generation
- Encryption for secrets

## Outputs
- MFA service
- Setup flow
- Verification flow
- Recovery codes

## Acceptance Criteria
- [ ] Generate TOTP secret
- [ ] Generate QR code for authenticator apps
- [ ] Verify TOTP codes
- [ ] Enable MFA after verification
- [ ] Generate recovery codes (10 codes)
- [ ] Verify recovery codes
- [ ] Disable MFA (with password)
- [ ] Encrypt secrets at rest
- [ ] Unit tests

## Implementation Details
Implement TOTP-based MFA using otplib. Generate secrets and QR codes. Verify 6-digit codes. Generate and store encrypted recovery codes. Support MFA in login flow.

## Files to Create
- packages/auth-service/src/mfa-service.ts
- packages/auth-service/src/utils/encryption.ts
- packages/auth-service/test/mfa-service.spec.ts

## Dependencies
Task 020-005