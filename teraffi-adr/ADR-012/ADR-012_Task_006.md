# Task: Embedding Freshness & Maintenance (3-4 hours)
**ADR:** ADR-012  
**Date:** 2025-10-01

## Inputs
- embedding_status table
- Entity update tracking

## Outputs
- Batch refresh job
- Stale embedding detection
- Rebuild utilities

## Acceptance Criteria
- [ ] Identify embeddings not updated in 30+ days
- [ ] Batch re-embedding job (nightly or weekly)
- [ ] Detect text changes via hash comparison
- [ ] Force refresh API endpoint (admin)
- [ ] Metrics on embedding freshness
- [ ] Dry-run mode for testing

## Implementation Details
Query embedding_status for stale entries. Compare text hashes to detect changes. Enqueue batch refresh jobs. Provide admin API to force re-embedding for specific entities.

## Files to Create
- packages/embedding-service/src/jobs/refresh.ts
- packages/embedding-service/src/freshness-checker.ts
- packages/api/src/routes/admin/embeddings.ts
- scripts/embedding-maintenance.ts

## Dependencies
Task 012-003