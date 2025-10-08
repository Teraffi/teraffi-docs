# Task: API Key Management (4-5 hours)
**ADR:** ADR-020  
**Date:** 2025-10-01

## Inputs
- API keys schema
- Secure key generation

## Outputs
- API key service
- Key generation
- Key validation

## Acceptance Criteria
- [ ] Generate API keys (prefixed format: teraffi_live_xxxxx)
- [ ] Hash keys before storage
- [ ] Store key prefix for identification
- [ ] Assign permissions to keys
- [ ] Set expiry dates
- [ ] Validate API keys in requests
- [ ] Track last_used_at
- [ ] Revoke keys
- [ ] Unit tests

## Implementation Details
Generate secure API keys with identifiable prefix. Hash before storage (never store plaintext). Support scoped permissions per key. Track usage. Allow revocation.

## Files to Create
- packages/auth-service/src/api-key-service.ts
- packages/auth-service/src/middleware/api-key-auth.ts
- packages/auth-service/test/api-key-service.spec.ts

## Dependencies
Task 020-001