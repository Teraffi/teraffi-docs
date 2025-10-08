# Task: Collaboration Service (Comments & Activity) (5-6 hours)
**ADR:** ADR-017  
**Date:** 2025-10-01

## Inputs
- User comments
- Deal activities
- WebSocket connections

## Outputs
- Collaboration service
- Comment threads
- Activity feed
- Real-time broadcasts

## Acceptance Criteria
- [ ] Add comments to deals
- [ ] Support threaded replies (parent_id)
- [ ] Extract and notify @mentions
- [ ] Log all activities (stage changes, uploads, comments)
- [ ] Real-time WebSocket broadcasts
- [ ] Activity feed with pagination
- [ ] Filter activities by type
- [ ] Unit and integration tests

## Implementation Details
Store comments and activities in PostgreSQL. Extract mentions using regex. Broadcast changes via WebSocket to connected clients. Maintain activity feed for transparency.

## Files to Create
- packages/deal-flow/src/collaboration-service.ts
- packages/deal-flow/src/websocket-manager.ts
- packages/deal-flow/test/collaboration.spec.ts

## Dependencies
Task 017-001, ADR-009 (Redis for WebSocket state)