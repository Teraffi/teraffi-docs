# Task: Publishable Key Issuance/Rotation Admin APIs (4-6 hours)
**ADR:** ADR-004  
**Date:** 2025-10-01

## Inputs
- Key Vault
- OpenAPI

## Outputs
- Admin endpoints

## Acceptance Criteria
- [ ] Create/rotate/list keys
- [ ] 24h overlap policy
- [ ] Audit logged

## Implementation Details
- Implement admin endpoints with Key Vault ops; update spec.

## Files to Create
- packages/api/src/routes/admin/keys.ts
- docs/api/openapi.v1.json

## Dependencies
Tasks 004-001..003
