# Task: Unsubscribe & Preference Center (4-5 hours)
**ADR:** ADR-022  
**Date:** 2025-10-01

## Inputs
- Legal requirements (CAN-SPAM, GDPR)
- User preferences

## Outputs
- Unsubscribe links
- Preference center UI
- One-click unsubscribe

## Acceptance Criteria
- [ ] Unsubscribe link in all emails
- [ ] One-click unsubscribe
- [ ] Preference center page
- [ ] Granular control per notification type
- [ ] List-Unsubscribe header (email)
- [ ] Re-subscribe option
- [ ] Audit trail of preference changes
- [ ] Compliance with CAN-SPAM/GDPR

## Implementation Details
Add unsubscribe links to emails. Build preference center UI where users can manage all notification settings. Support one-click unsubscribe. Maintain audit trail for compliance.

## Files to Create
- packages/api/src/routes/unsubscribe.ts
- apps/web/src/pages/preferences/notifications.tsx
- packages/notifications/src/unsubscribe-handler.ts

## Dependencies
Task 022-010, Task 022-013