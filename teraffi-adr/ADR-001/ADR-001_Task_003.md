# Task: OpenAPI 3.1 Spec Generator & Drift Check (3-4 hours)
**ADR:** ADR-001  
**Date:** 2025-10-01

## Inputs
- Route registry
- CI

## Outputs
- openapi.v1.json
- CI script to fail on drift

## Acceptance Criteria
- [ ] Spec generated
- [ ] CI fails on drift
- [ ] Endpoints documented

## Implementation Details
- Generate spec; store in docs/api; add CI step to compare.

## Files to Create
- docs/api/openapi.v1.json
- scripts/ci/check-openapi.ts

## Dependencies
Task 001-002
