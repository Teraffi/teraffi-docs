# Task: Alert Engine (4-6 hours)
**ADR:** ADR-016  
**Date:** 2025-10-01

## Inputs
- Trend data from Neo4j
- Alert thresholds
- Notification service

## Outputs
- Alert engine
- Threshold monitoring
- Entity notification

## Acceptance Criteria
- [ ] Detect emerging trends (momentum >0.3)
- [ ] Detect accelerating trends (velocity >0.5)
- [ ] Detect peak warnings (predicted peak <30 days)
- [ ] Detect declining trends (velocity <0)
- [ ] Find affected entities (aligned with trends)
- [ ] Send notifications with priority levels
- [ ] Daily batch job
- [ ] Unit tests

## Implementation Details
Query Neo4j for trends meeting alert thresholds. Find entities with ALIGNS_WITH relationships to alerting trends. Send notifications via notification service. Run as scheduled job (daily).

## Files to Create
- packages/social-listening/src/alert-engine.ts
- packages/social-listening/src/jobs/check-alerts.ts
- packages/social-listening/test/alert-engine.spec.ts
- db/migrations/XXX_create_trend_alerts.sql

## Dependencies
Task 016-008, ADR-011 (Neo4j)