# Task: Database Schema & Core Models (3-4 hours)
**ADR:** ADR-017  
**Date:** 2025-10-01

## Inputs
- Schema requirements from ADR-017
- Relationship to existing tables

## Outputs
- Database migrations
- TypeScript models/types
- Foreign key relationships

## Acceptance Criteria
- [ ] Create partnerships table with all fields
- [ ] Create deal_documents table
- [ ] Create deal_comments table
- [ ] Create deal_activities table
- [ ] Create deal_audit_log table
- [ ] Create deal_milestones table
- [ ] All indexes created
- [ ] Foreign keys and cascades configured
- [ ] TypeScript types generated

## Implementation Details
Create PostgreSQL migrations for all deal flow tables. Define proper indexes for common query patterns. Set up cascading deletes where appropriate. Generate TypeScript types from schema.

## Files to Create
- db/migrations/XXX_create_partnerships.sql
- db/migrations/XXX_create_deal_documents.sql
- db/migrations/XXX_create_deal_comments.sql
- db/migrations/XXX_create_deal_activities.sql
- db/migrations/XXX_create_deal_audit_log.sql
- db/migrations/XXX_create_deal_milestones.sql
- packages/deal-flow/src/types/models.ts

## Dependencies
ADR-007 (PostgreSQL)