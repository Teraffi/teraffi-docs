# Task: API Contract Comprehensive Test Suite (CRUD, pagination, errors) (6-8 hours)
**ADR:** ADR-001  
**Date:** 2025-10-01

## Inputs
- Fastify server
- OpenAPI

## Outputs
- Test suite covering 30+ cases

## Acceptance Criteria
- [ ] All scenarios covered
- [ ] OpenAPI remains in sync
- [ ] CI green

## Implementation Details
- Group prior granular scenarios into a single suite; snapshot responses; ensure docs updated.

## Files to Create
- packages/api/test/api.contract.spec.ts

## Dependencies
Tasks 001-001..001-006
