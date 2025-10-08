# Task: Push Notification Service (5-6 hours)
**ADR:** ADR-022  
**Date:** 2025-10-01

## Inputs
- Firebase Cloud Messaging credentials
- APNs credentials (iOS)
- Device token management

## Outputs
- Push notification service
- Multi-platform support
- Device token management
- Delivery tracking

## Acceptance Criteria
- [ ] Send push to iOS (APNs)
- [ ] Send push to Android (FCM)
- [ ] Store device tokens
- [ ] Handle token expiration
- [ ] Track delivery status
- [ ] Support notification actions
- [ ] Badge count updates
- [ ] Rich notifications (images, actions)
- [ ] Unit tests

## Implementation Details
Integrate with Firebase Cloud Messaging and APNs. Store device tokens per user. Send push notifications with proper formatting per platform. Track delivery and handle failures.

## Files to Create
- packages/notifications/src/channels/push-service.ts
- packages/notifications/src/device-manager.ts
- db/migrations/XXX_create_user_devices.sql
- packages/notifications/test/push-service.spec.ts

## Dependencies
Task 022-002