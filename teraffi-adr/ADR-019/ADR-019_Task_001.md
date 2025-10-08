# Task: Database Schema - Add Tenant Infrastructure (4-5 hours)
**ADR:** ADR-019  
**Date:** 2025-10-01

## Inputs
- Existing database schema
- Tenant requirements

## Outputs
- Tenants table
- tenant_id columns on all tables
- Indexes for performance
- Audit log table

## Acceptance Criteria
- [ ] Create tenants table with all fields
- [ ] Add tenant_id UUID column to all existing tables
- [ ] Create indexes on tenant_id columns
- [ ] Create tenant_audit_log table
- [ ] Foreign key constraints to tenants table
- [ ] Migration scripts (up/down)
- [ ] No data loss during migration
- [ ] Documentation

## Implementation Details
Create tenants table. Add tenant_id column to members, brands, partnerships, deal_documents, and all other tables. Create indexes for query performance. Set up audit log table for compliance.

## Files to Create
- db/migrations/XXX_create_tenants_table.sql
- db/migrations/XXX_add_tenant_id_columns.sql
- db/migrations/XXX_create_tenant_indexes.sql
- db/migrations/XXX_create_tenant_audit_log.sql
- docs/database/tenant_schema.md

## Dependencies
ADR-007 (PostgreSQL)