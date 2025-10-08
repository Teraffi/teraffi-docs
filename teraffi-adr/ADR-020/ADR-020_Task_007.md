# Task: Role-Based Access Control (RBAC) System (5-6 hours)
**ADR:** ADR-020  
**Date:** 2025-10-01

## Inputs
- Roles schema
- Permission definitions

## Outputs
- RBAC service
- Permission checking
- Role assignment
- System roles

## Acceptance Criteria
- [ ] Create/update/delete roles
- [ ] Assign permissions to roles
- [ ] Assign roles to users
- [ ] Check user permissions
- [ ] Support wildcard permissions (e.g., admin:*)
- [ ] Load user roles and permissions efficiently
- [ ] Define system roles (cannot be deleted)
- [ ] Unit tests

## Implementation Details
Build RBAC system with roles and permissions. Support hierarchical permissions with wildcards. Cache user permissions for performance. Define system roles for platform, tenant admin, etc.

## Files to Create
- packages/auth-service/src/rbac-service.ts
- packages/auth-service/src/permissions.ts
- packages/auth-service/src/system-roles.ts
- packages/auth-service/test/rbac-service.spec.ts

## Dependencies
Task 020-001