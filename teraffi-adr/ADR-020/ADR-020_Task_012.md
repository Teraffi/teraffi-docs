# Task: SSO Integration (SAML/OAuth) (8-10 hours)
**ADR:** ADR-020  
**Date:** 2025-10-01

## Inputs
- SAML/OAuth libraries
- Identity provider configurations

## Outputs
- SSO service
- SAML handler
- OAuth handler
- Just-in-time provisioning

## Acceptance Criteria
- [ ] SAML 2.0 authentication
- [ ] OAuth 2.0 / OpenID Connect
- [ ] Configure per tenant
- [ ] Just-in-time user provisioning
- [ ] Map SSO attributes to user fields
- [ ] Support multiple IdPs per tenant
- [ ] Handle SSO errors gracefully
- [ ] Integration tests

## Implementation Details
Implement SAML and OAuth handlers. Support tenant-specific IdP configuration. Create users on first SSO login (JIT provisioning). Map external attributes to user profile.

## Files to Create
- packages/auth-service/src/sso/saml-handler.ts
- packages/auth-service/src/sso/oauth-handler.ts
- packages/auth-service/src/sso/jit-provisioning.ts
- packages/auth-service/test/sso.spec.ts

## Dependencies
Tasks 020-004, 020-005