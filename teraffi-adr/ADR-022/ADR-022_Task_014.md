# Task: Notification Triggers Integration (5-6 hours)
**ADR:** ADR-022  
**Date:** 2025-10-01

## Inputs
- Existing services (Deal Flow, Trends, etc.)
- Notification requirements per feature

## Outputs
- Notification triggers
- Event handlers
- Integration points

## Acceptance Criteria
- [ ] Deal stage change notifications
- [ ] Partnership proposal notifications
- [ ] Milestone completion notifications
- [ ] Trend alert notifications
- [ ] @mention notifications
- [ ] Document upload notifications
- [ ] Contract signature notifications
- [ ] Comment notifications
- [ ] Integration tests

## Implementation Details
Integrate notification service with existing features. Add notification triggers at key events. Use appropriate templates and priorities. Test all notification paths.

## Files to Create
- packages/deal-flow/src/notifications/deal-notifications.ts
- packages/social-listening/src/notifications/trend-notifications.ts
- packages/deal-flow/src/notifications/collaboration-notifications.ts
- packages/notifications/test/integrations.spec.ts

## Dependencies
Task 022-002, ADR-017 (Deal Flow), ADR-016 (Social Listening)