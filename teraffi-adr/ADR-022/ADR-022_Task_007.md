# Task: Slack Integration (4-5 hours)
**ADR:** ADR-022  
**Date:** 2025-10-01

## Inputs
- Slack API credentials
- Webhook URLs
- Slack workspace tokens

## Outputs
- Slack notification service
- OAuth flow for workspace connection
- Channel/DM delivery

## Acceptance Criteria
- [ ] Send to Slack channels
- [ ] Send to Slack DMs
- [ ] OAuth flow for workspace connection
- [ ] Store Slack tokens per user/tenant
- [ ] Format messages with Slack blocks
- [ ] Support buttons and actions
- [ ] Handle rate limits
- [ ] Unit tests

## Implementation Details
Integrate with Slack API. Support OAuth for workspace connection. Send formatted messages to channels or DMs. Use Slack Block Kit for rich formatting. Handle rate limiting.

## Files to Create
- packages/notifications/src/channels/slack-service.ts
- packages/notifications/src/oauth/slack-oauth.ts
- packages/notifications/test/slack-service.spec.ts

## Dependencies
Task 022-002