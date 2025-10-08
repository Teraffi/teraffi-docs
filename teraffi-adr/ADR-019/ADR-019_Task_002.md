# Task: Tenant Context Middleware & Propagation (5-6 hours)
**ADR:** ADR-019  
**Date:** 2025-10-01

## Inputs
- JWT authentication
- Async local storage
- Request lifecycle

## Outputs
- Tenant context middleware
- Context propagation system
- Tenant extraction from JWT

## Acceptance Criteria
- [ ] Extract tenant_id from JWT
- [ ] Load tenant metadata (tier, quotas, features)
- [ ] Store in request.tenantContext
- [ ] Propagate via async local storage
- [ ] Available in all downstream services
- [ ] Handle missing/invalid tenant gracefully
- [ ] Performance overhead <5ms
- [ ] Unit tests

## Implementation Details
Create middleware to extract tenant from JWT. Load tenant metadata from registry. Store in request context and async local storage for propagation. Validate tenant is active.

## Files to Create
- packages/core/src/middleware/tenant-context.ts
- packages/core/src/context/async-storage.ts
- packages/core/test/tenant-context.spec.ts

## Dependencies
Task 019-001, ADR-001 (API)