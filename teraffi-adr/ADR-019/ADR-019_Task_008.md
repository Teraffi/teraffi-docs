# Task: Resource Quotas & Rate Limiting (5-6 hours)
**ADR:** ADR-019  
**Date:** 2025-10-01

## Inputs
- Tenant quotas
- Request metrics
- Redis for counters

## Outputs
- Quota enforcement middleware
- Rate limiting
- Usage tracking

## Acceptance Criteria
- [ ] Enforce API requests per hour
- [ ] Enforce partnerships per month
- [ ] Enforce storage limits
- [ ] Enforce team member limits
- [ ] Return 429 when limits exceeded
- [ ] Rate limit headers (X-RateLimit-*)
- [ ] Dashboard for quota usage
- [ ] Unit tests

## Implementation Details
Implement middleware to track and enforce quotas. Use Redis counters with expiry for rate windows. Check quotas before allowing resource creation. Return proper HTTP status codes and headers.

## Files to Create
- packages/core/src/middleware/quota-enforcement.ts
- packages/tenant-management/src/quota-tracker.ts
- packages/core/test/quota-enforcement.spec.ts

## Dependencies
Tasks 019-002, 019-003, 019-007