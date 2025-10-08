# Task: Slack Alert Integration (3-4 hours)
**ADR:** ADR-024  
**Date:** 2025-10-01

## Inputs
- Slack workspace
- Webhook URLs
- Channel structure

## Outputs
- Slack alert receiver
- Channel routing
- Alert formatting

## Acceptance Criteria
- [ ] Configure Slack webhook
- [ ] Add receiver to Alertmanager
- [ ] Route alerts to appropriate channels
- [ ] Format alerts with details
- [ ] Color-code by severity
- [ ] Include runbook links
- [ ] Test all severity levels
- [ ] Documentation

## Implementation Details
Send alerts to Slack channels. Route critical to #alerts-critical, warnings to #alerts-warnings. Format with clear information and runbook links.

## Files to Create
- infrastructure/alertmanager/receivers/slack.yml
- docs/oncall/slack_alerts.md

## Dependencies
Task 024-016