# Task: Observability: tenant_scope.missing metric + dashboard (2-3 hours)
**ADR:** ADR-006  
**Date:** 2025-10-01

## Inputs
- Telemetry lib

## Outputs
- Metric emit + doc

## Acceptance Criteria
- [ ] Metric emitted on violations
- [ ] Dashboard created
- [ ] Alert set

## Implementation Details
- Emit metric in guard; define dashboard panel.

## Files to Create
- packages/ops/src/telemetry.ts
- docs/observability/tenant_guard.md

## Dependencies
Task 006-001
