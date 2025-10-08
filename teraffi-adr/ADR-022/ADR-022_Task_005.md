# Task: In-App Notifications (WebSocket) (5-6 hours)
**ADR:** ADR-022  
**Date:** 2025-10-01

## Inputs
- WebSocket infrastructure
- Real-time requirements

## Outputs
- WebSocket notification delivery
- Connection management
- Real-time updates
- Notification badge updates

## Acceptance Criteria
- [ ] Send notifications via WebSocket
- [ ] Track connected users
- [ ] Broadcast to specific user's connections
- [ ] Handle disconnections gracefully
- [ ] Send unread count updates
- [ ] Support notification actions (mark read via WebSocket)
- [ ] Reconnection handling
- [ ] Unit tests

## Implementation Details
Use WebSocket to deliver real-time in-app notifications. Track user connections. Broadcast notifications to all active sessions. Update unread counts in real-time. Handle connection lifecycle.

## Files to Create
- packages/notifications/src/channels/in-app-service.ts
- packages/notifications/src/websocket/notification-broadcaster.ts
- packages/notifications/test/in-app-service.spec.ts

## Dependencies
Task 022-002, ADR-001 (WebSocket)