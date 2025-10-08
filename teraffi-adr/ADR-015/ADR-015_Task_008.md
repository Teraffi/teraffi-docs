# Task: Authenticity Detection Component (4-5 hours)
**ADR:** ADR-015  
**Date:** 2025-10-01

## Inputs
- Entity data from Neo4j
- Historical trend alignment
- LLM for analysis

## Outputs
- Authenticity calculator
- Genuine vs opportunistic detection
- Reasoning and red flags

## Acceptance Criteria
- [ ] Gather entity evidence (values, actions, history)
- [ ] LLM analysis of authenticity
- [ ] Return score 0.0-1.0
- [ ] Provide reasoning text
- [ ] Identify red flags
- [ ] Store analysis for explainability
- [ ] Unit tests with authentic/opportunistic examples

## Implementation Details
Query Neo4j for entity properties and trend history. Use LLM to assess if alignment is genuine (consistent values/actions) or opportunistic (trend-jumping). Store reasoning for transparency.

## Files to Create
- packages/affinity-engine/src/components/authenticity.ts
- db/migrations/XXX_create_authenticity_analyses.sql
- packages/affinity-engine/test/authenticity.spec.ts

## Dependencies
Task 015-003, ADR-011 (Neo4j), ADR-014 (LLM Gateway)