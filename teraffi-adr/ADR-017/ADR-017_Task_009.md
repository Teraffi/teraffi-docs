# Task: Notification System Integration (3-4 hours)
**ADR:** ADR-017  
**Date:** 2025-10-01

## Inputs
- Notification templates
- Recipient configuration
- Notification service

## Outputs
- Notification dispatcher
- Email/in-app notifications
- Template rendering

## Acceptance Criteria
- [ ] Define notification templates per transition
- [ ] Send to correct recipients (entity1, entity2, both, team)
- [ ] Email notifications
- [ ] In-app notifications
- [ ] @mention notifications
- [ ] Milestone completion notifications
- [ ] Template variable substitution
- [ ] Unit tests

## Implementation Details
Create notification templates for each deal event. Determine recipients based on transition rules. Send via email and in-app channels. Support variable substitution in templates.

## Files to Create
- packages/deal-flow/src/notification-dispatcher.ts
- packages/deal-flow/src/templates/notifications.ts
- packages/deal-flow/test/notifications.spec.ts

## Dependencies
Task 017-002, Notification service