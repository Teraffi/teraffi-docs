# Task: Template Engine & Management (4-5 hours)
**ADR:** ADR-022  
**Date:** 2025-10-01

## Inputs
- Template requirements
- Variable substitution needs
- Multi-channel formats

## Outputs
- Template engine
- Template CRUD
- Variable interpolation
- Default templates

## Acceptance Criteria
- [ ] Template storage and retrieval
- [ ] Variable substitution ({{variable}} syntax)
- [ ] Support email (HTML + text)
- [ ] Support in-app format
- [ ] Support push notification format
- [ ] Support Slack format
- [ ] Tenant-specific templates
- [ ] Default system templates
- [ ] Unit tests

## Implementation Details
Build template engine supporting multiple formats. Implement variable interpolation. Store templates per notification type and channel. Support tenant customization with fallback to system templates.

## Files to Create
- packages/notifications/src/template-engine.ts
- packages/notifications/src/template-manager.ts
- packages/notifications/src/templates/defaults.ts
- packages/notifications/test/template-engine.spec.ts

## Dependencies
Task 022-001