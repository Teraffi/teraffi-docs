# Task: Temporal Worker Bootstrap & Health (3-4 hours)
**ADR:** ADR-002  
**Date:** 2025-10-01

## Inputs
- Temporal SDK
- Namespace creds

## Outputs
- Worker process
- Health log

## Acceptance Criteria
- [ ] Connects to cloud
- [ ] Registers wf/activities
- [ ] Graceful shutdown

## Implementation Details
- Start worker; add health endpoint/log; wire basic sample.

## Files to Create
- packages/worker/src/index.ts
- packages/worker/src/workflows/sample.workflow.ts

## Dependencies
ADR-001 tasks
