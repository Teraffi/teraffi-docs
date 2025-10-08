# Task: Email Delivery Service (4-5 hours)
**ADR:** ADR-022  
**Date:** 2025-10-01

## Inputs
- Email provider credentials (SendGrid/AWS SES)
- Email templates
- Delivery tracking requirements

## Outputs
- Email service
- HTML/text rendering
- Delivery tracking
- Bounce handling

## Acceptance Criteria
- [ ] Send transactional emails
- [ ] Render HTML templates
- [ ] Generate plain text fallback
- [ ] Track delivery status
- [ ] Handle bounces and complaints
- [ ] Retry on transient failures
- [ ] Support attachments (optional)
- [ ] Unsubscribe link handling
- [ ] Unit tests with mocked provider

## Implementation Details
Integrate with SendGrid or AWS SES. Render email templates. Track message IDs for delivery confirmation. Handle webhooks for bounce/complaint notifications. Implement retry logic.

## Files to Create
- packages/notifications/src/channels/email-service.ts
- packages/notifications/src/webhooks/email-webhook-handler.ts
- packages/notifications/test/email-service.spec.ts

## Dependencies
Task 022-003