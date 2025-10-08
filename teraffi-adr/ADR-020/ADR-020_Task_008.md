# Task: Authorization Middleware (3-4 hours)
**ADR:** ADR-020  
**Date:** 2025-10-01

## Inputs
- JWT verification
- RBAC service
- Request context

## Outputs
- Auth middleware
- Permission checks
- Route protection

## Acceptance Criteria
- [ ] Verify JWT on protected routes
- [ ] Extract user from token
- [ ] Store in request.user
- [ ] requireAuth middleware
- [ ] requirePermission(permission) middleware
- [ ] requireRole(role) middleware
- [ ] Return 401 if not authenticated
- [ ] Return 403 if not authorized
- [ ] Integration tests

## Implementation Details
Create Fastify middleware to verify JWT and check permissions. Extract user info from token. Provide decorators for route protection. Support both authentication and authorization checks.

## Files to Create
- packages/auth-service/src/middleware/auth.ts
- packages/auth-service/src/middleware/authorization.ts
- packages/auth-service/test/middleware.spec.ts

## Dependencies
Tasks 020-003, 020-007