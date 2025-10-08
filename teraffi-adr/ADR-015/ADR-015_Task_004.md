# Task: Goal Extraction & Tracking (4-6 hours)
**ADR:** ADR-015  
**Date:** 2025-10-01

## Inputs
- "What do you want?" natural language input
- LLM Gateway
- PostgreSQL for storage

## Outputs
- Goal extraction service
- Structured goal objects
- Time-stamped goal tracking

## Acceptance Criteria
- [ ] Parse goals from natural language
- [ ] Categorize: market_expansion, revenue_growth, brand_elevation, ip_monetization
- [ ] Extract target demographics, geography, values
- [ ] Store with timestamps in PostgreSQL
- [ ] Support goal updates (track evolution)
- [ ] Temporal weighting (exponential decay)
- [ ] Unit tests with goal examples

## Implementation Details
Use LLM to extract structured goals from user input. Store in database with timestamps for temporal analysis. Implement exponential decay weighting for goal relevance over time.

## Files to Create
- packages/affinity-engine/src/goal-extraction.ts
- packages/affinity-engine/src/types/goals.ts
- db/migrations/XXX_create_member_goals.sql
- packages/affinity-engine/test/goal-extraction.spec.ts

## Dependencies
Task 015-001, ADR-014 (LLM Gateway)