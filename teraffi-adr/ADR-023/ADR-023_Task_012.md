# Task: Billing REST API (4-5 hours)
**ADR:** ADR-023  
**Date:** 2025-10-01

## Inputs
- All billing services
- API specifications

## Outputs
- REST endpoints
- Request validation
- OpenAPI documentation

## Acceptance Criteria
- [ ] POST /api/v1/billing/checkout - create checkout session
- [ ] POST /api/v1/billing/portal - get portal URL
- [ ] GET /api/v1/billing/subscription - get current subscription
- [ ] PUT /api/v1/billing/subscription - update subscription
- [ ] DELETE /api/v1/billing/subscription - cancel subscription
- [ ] GET /api/v1/billing/invoices - list invoices
- [ ] GET /api/v1/billing/usage - current usage
- [ ] POST /webhooks/stripe - webhook endpoint
- [ ] Request validation
- [ ] OpenAPI spec
- [ ] API tests

## Implementation Details
Expose billing services via REST API. Validate requests. Handle authentication. Document with OpenAPI. Include webhook endpoint for Stripe events.

## Files to Create
- packages/api/src/routes/billing.ts
- packages/api/src/webhooks/stripe.ts
- packages/api/test/billing.e2e.spec.ts
- docs/api/openapi-billing.yaml

## Dependencies
Tasks 023-004 through 023-011