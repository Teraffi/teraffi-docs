# Task: Audit Logging & Security Events (3-4 hours)
**ADR:** ADR-020  
**Date:** 2025-10-01

## Inputs
- Auth events
- Compliance requirements

## Outputs
- Audit logger
- Security event tracking
- Query interface

## Acceptance Criteria
- [ ] Log all authentication events (login, logout, failed attempts)
- [ ] Log authorization failures
- [ ] Log password changes
- [ ] Log MFA events
- [ ] Log role/permission changes
- [ ] Include IP address, user agent
- [ ] Immutable audit log
- [ ] Query interface for security team
- [ ] Integration tests

## Implementation Details
Create audit logging service. Log all security-relevant events with metadata. Store in append-only table. Provide query interface for security investigations.

## Files to Create
- packages/auth-service/src/audit-logger.ts
- packages/auth-service/src/queries/audit-queries.ts
- packages/auth-service/test/audit-logger.spec.ts

## Dependencies
Task 020-001