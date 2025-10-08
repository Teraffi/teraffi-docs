# Task: Session Management & Monitoring (3-4 hours)
**ADR:** ADR-020  
**Date:** 2025-10-01

## Inputs
- Active sessions
- Device information

## Outputs
- Session tracking
- Active sessions list
- Session revocation

## Acceptance Criteria
- [ ] Track active sessions per user
- [ ] Store device info (user agent, IP)
- [ ] List user's active sessions
- [ ] Revoke individual sessions
- [ ] Revoke all sessions (forced logout)
- [ ] Session timeout enforcement
- [ ] Notification on new device login
- [ ] Unit tests

## Implementation Details
Track active sessions using refresh tokens. Store device metadata. Allow users to view and revoke sessions. Notify on suspicious activity.

## Files to Create
- packages/auth-service/src/session-manager.ts
- packages/auth-service/test/session-manager.spec.ts

## Dependencies
Task 020-006