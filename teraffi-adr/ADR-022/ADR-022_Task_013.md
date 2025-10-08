# Task: Notification REST API (4-5 hours)
**ADR:** ADR-022  
**Date:** 2025-10-01

## Inputs
- Notification service
- Preference service
- API specifications

## Outputs
- REST endpoints
- WebSocket endpoints
- OpenAPI documentation

## Acceptance Criteria
- [ ] GET /api/v1/notifications - list notifications
- [ ] GET /api/v1/notifications/unread-count
- [ ] PUT /api/v1/notifications/:id/read - mark as read
- [ ] PUT /api/v1/notifications/mark-all-read
- [ ] DELETE /api/v1/notifications/:id - archive
- [ ] GET /api/v1/notifications/preferences
- [ ] PUT /api/v1/notifications/preferences/:type
- [ ] Request validation
- [ ] Pagination
- [ ] OpenAPI spec
- [ ] API tests

## Implementation Details
Expose notification services via REST API. Provide endpoints for listing, marking as read, managing preferences. Support pagination. Document with OpenAPI.

## Files to Create
- packages/api/src/routes/notifications.ts
- packages/api/test/notifications.e2e.spec.ts
- docs/api/openapi-notifications.yaml

## Dependencies
Tasks 022-002, 022-010