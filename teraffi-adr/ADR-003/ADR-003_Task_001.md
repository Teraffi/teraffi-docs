# Task: Outbox Table DDL + Repository (3-4 hours)
**ADR:** ADR-003  
**Date:** 2025-10-01

## Inputs
- Postgres

## Outputs
- DDL + repo

## Acceptance Criteria
- [ ] Table created
- [ ] CRUD methods
- [ ] Unit tests

## Implementation Details
- Create outbox table with attempts/published_at; repo methods.

## Files to Create
- db/migrations/020_outbox.sql
- packages/api/src/repos/outbox.repo.ts
- packages/api/test/outbox.repo.spec.ts

## Dependencies
DB ready
