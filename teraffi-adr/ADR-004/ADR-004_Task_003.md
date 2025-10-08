# Task: Webhook Signature Verification Utility (3-4 hours)
**ADR:** ADR-004  
**Date:** 2025-10-01

## Inputs
- HMAC secret
- Clock skew tolerance

## Outputs
- Verifier util + tests

## Acceptance Criteria
- [ ] Rejects invalid/old
- [ ] Accepts valid
- [ ] Docs updated

## Implementation Details
- Implement Stripe-like verifier; document usage.

## Files to Create
- packages/api/src/lib/webhookVerify.ts
- packages/api/test/webhookVerify.spec.ts

## Dependencies
Task 004-002
