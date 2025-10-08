# Task: Fastify API Bootstrap + AJV Validation (3-4 hours)
**ADR:** ADR-001  
**Date:** 2025-10-01

## Inputs
- Fastify
- @fastify/ajv-compiler

## Outputs
- Server factory
- Health endpoint
- Validation plugin

## Acceptance Criteria
- [ ] Server boots
- [ ] /healthz=200
- [ ] 400 on invalid body

## Implementation Details
- Build server with logger + AJV; add sample route + schema.

## Files to Create
- packages/api/src/index.ts
- packages/api/src/plugins/validation.ts
- packages/api/src/routes/example.ts

## Dependencies
Task 001-001
