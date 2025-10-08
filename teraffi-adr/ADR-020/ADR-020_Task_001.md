# Task: Database Schema - Authentication Tables (3-4 hours)
**ADR:** ADR-020  
**Date:** 2025-10-01

## Inputs
- Authentication requirements
- User and role models

## Outputs
- Users table
- Roles and permissions tables
- API keys table
- Refresh tokens table
- Auth audit log table

## Acceptance Criteria
- [ ] Create users table with all fields
- [ ] Create roles table
- [ ] Create user_roles junction table
- [ ] Create api_keys table
- [ ] Create refresh_tokens table
- [ ] Create auth_audit_log table
- [ ] All indexes created
- [ ] Foreign key constraints
- [ ] Migration scripts (up/down)
- [ ] Documentation

## Implementation Details
Create PostgreSQL tables for user authentication. Include MFA fields, SSO fields, account lockout. Define roles and permissions structure. Set up refresh token storage and audit logging.

## Files to Create
- db/migrations/XXX_create_users_table.sql
- db/migrations/XXX_create_roles_tables.sql
- db/migrations/XXX_create_api_keys_table.sql
- db/migrations/XXX_create_refresh_tokens_table.sql
- db/migrations/XXX_create_auth_audit_log.sql
- docs/database/auth_schema.md

## Dependencies
ADR-019 (Multi-Tenancy)