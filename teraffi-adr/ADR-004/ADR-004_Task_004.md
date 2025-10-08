# Task: Route Guards (Scopes & Roles) (3-4 hours)
**ADR:** ADR-004  
**Date:** 2025-10-01

## Inputs
- JWT claims
- Guard helper

## Outputs
- Middleware + tests

## Acceptance Criteria
- [ ] Deny without scope
- [ ] Allow with scope
- [ ] Audit log entry

## Implementation Details
- Implement `requireScopes([...])`; integrate in sample routes.

## Files to Create
- packages/api/src/middleware/guard.ts
- packages/api/test/guard.spec.ts

## Dependencies
Tasks 004-001..003
