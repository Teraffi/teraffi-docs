# Task: Notification Testing Tools (3-4 hours)
**ADR:** ADR-022  
**Date:** 2025-10-01

## Inputs
- Testing requirements
- QA needs

## Outputs
- Test notification sender
- Preview tool
- Admin testing UI

## Acceptance Criteria
- [ ] Admin UI to send test notifications
- [ ] Preview notifications before sending
- [ ] Test all channels
- [ ] Populate with sample data
- [ ] Send to test users/devices
- [ ] Template preview
- [ ] Verify rendering across channels

## Implementation Details
Build admin tools for testing notifications. Allow sending test notifications to specific users. Preview templates with sample data. Verify delivery across all channels.

## Files to Create
- apps/admin/src/pages/notifications/test.tsx
- packages/notifications/src/testing/test-sender.ts
- packages/notifications/src/testing/preview-generator.ts

## Dependencies
Tasks 022-002, 022-009