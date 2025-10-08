# Task: Orchestration & Scheduling (3-4 hours)
**ADR:** ADR-016  
**Date:** 2025-10-01

## Inputs
- All collector services
- Processing pipeline
- Cron scheduler

## Outputs
- Main orchestration service
- Scheduled jobs
- Error handling

## Acceptance Criteria
- [ ] Orchestrate collection → processing → extraction → update flow
- [ ] Schedule hourly collection jobs
- [ ] Schedule daily trend extraction
- [ ] Schedule daily alert checks
- [ ] Error handling and retry logic
- [ ] Metrics emission at each stage
- [ ] Integration tests end-to-end

## Implementation Details
Wire together all services into coordinated pipeline. Use cron or scheduler for periodic jobs. Implement error handling with dead letter queues. Emit metrics for monitoring.

## Files to Create
- packages/social-listening/src/orchestrator.ts
- packages/social-listening/src/scheduler.ts
- packages/social-listening/test/integration.spec.ts

## Dependencies
Tasks 016-001 through 016-009