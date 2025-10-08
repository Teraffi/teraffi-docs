# Task: PagerDuty Integration (3-4 hours)
**ADR:** ADR-024  
**Date:** 2025-10-01

## Inputs
- PagerDuty account
- Service keys
- On-call schedules

## Outputs
- PagerDuty integration
- Incident management
- On-call routing

## Acceptance Criteria
- [ ] Configure PagerDuty service
- [ ] Add integration key to Alertmanager
- [ ] Test critical alert delivery
- [ ] Configure escalation policies
- [ ] Set up on-call schedules
- [ ] Configure incident auto-resolution
- [ ] Test alert acknowledgment
- [ ] Documentation

## Implementation Details
Integrate Alertmanager with PagerDuty. Send critical alerts to on-call engineers. Configure escalation if no acknowledgment. Support auto-resolution when alerts clear.

## Files to Create
- infrastructure/alertmanager/receivers/pagerduty.yml
- docs/oncall/pagerduty_setup.md

## Dependencies
Task 024-016