# Task: Webhook Delivery Service (3-4 hours)
**ADR:** ADR-022  
**Date:** 2025-10-01

## Inputs
- User-configured webhook URLs
- Security requirements

## Outputs
- Webhook service
- HMAC signature
- Retry logic

## Acceptance Criteria
- [ ] POST notification JSON to webhook URL
- [ ] HMAC signature for verification
- [ ] Timeout handling (10 seconds)
- [ ] Retry on failure (exponential backoff)
- [ ] Track delivery status
- [ ] Support custom headers
- [ ] Disable after repeated failures
- [ ] Unit tests

## Implementation Details
Send HTTP POST requests to user-configured URLs. Sign with HMAC for security. Implement retries with exponential backoff. Track success/failure. Disable webhooks after multiple failures.

## Files to Create
- packages/notifications/src/channels/webhook-service.ts
- packages/notifications/src/security/hmac-signer.ts
- packages/notifications/test/webhook-service.spec.ts

## Dependencies
Task 022-002