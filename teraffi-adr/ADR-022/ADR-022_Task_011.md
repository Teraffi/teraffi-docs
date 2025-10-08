# Task: Batching & Digest Service (5-6 hours)
**ADR:** ADR-022  
**Date:** 2025-10-01

## Inputs
- User preferences
- Batch keys
- Scheduling requirements

## Outputs
- Digest generator
- Batch processor
- Scheduled jobs

## Acceptance Criteria
- [ ] Group notifications by batch_key
- [ ] Generate hourly digests
- [ ] Generate daily digests
- [ ] Generate weekly digests
- [ ] Render digest email templates
- [ ] Mark batched notifications as sent
- [ ] Schedule digest jobs
- [ ] Handle empty digests (don't send)
- [ ] Unit tests

## Implementation Details
Build service to batch notifications based on user frequency preferences. Generate digest emails grouping related notifications. Schedule jobs for hourly/daily/weekly digests. Render attractive digest templates.

## Files to Create
- packages/notifications/src/digest-service.ts
- packages/notifications/src/jobs/generate-digests.ts
- packages/notifications/src/templates/digest-template.ts
- packages/notifications/test/digest-service.spec.ts

## Dependencies
Tasks 022-004, 022-009, 022-010