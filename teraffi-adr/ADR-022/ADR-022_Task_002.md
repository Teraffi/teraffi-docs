# Task: Core Notification Service (5-6 hours)
**ADR:** ADR-022  
**Date:** 2025-10-01

## Inputs
- Notification schema
- Queue infrastructure
- Business requirements

## Outputs
- Notification service
- CRUD operations
- Queue integration
- Priority handling

## Acceptance Criteria
- [ ] Create notification
- [ ] Batch create notifications
- [ ] Mark as read/unread
- [ ] Mark all as read
- [ ] Get unread count
- [ ] List notifications (with pagination)
- [ ] Archive notifications
- [ ] Queue for processing
- [ ] Priority-based queueing
- [ ] Unit tests

## Implementation Details
Build core notification service handling CRUD operations. Queue notifications for async processing. Support priority levels. Track read/unread status. Provide list and count methods.

## Files to Create
- packages/notifications/src/notification-service.ts
- packages/notifications/src/types/notification.ts
- packages/notifications/test/notification-service.spec.ts

## Dependencies
Task 022-001, ADR-003 (Queue)