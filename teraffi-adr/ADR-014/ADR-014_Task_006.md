# Task: Cost Tracking System (4-5 hours)
**ADR:** ADR-014  
**Date:** 2025-10-01

## Inputs
- PostgreSQL connection
- Provider pricing tables
- Token usage data

## Outputs
- Cost tracker service
- Database schema
- Cost calculation logic
- Reporting queries

## Acceptance Criteria
- [ ] Record every LLM request with cost
- [ ] Calculate costs based on provider/model pricing
- [ ] Track by provider, model, use_case, tenant
- [ ] Query daily/monthly costs
- [ ] Cost breakdown by use case
- [ ] Monthly partition tables
- [ ] Unit tests for cost calculations

## Implementation Details
Maintain pricing tables for each provider/model. Calculate costs from token usage. Store in partitioned PostgreSQL table. Provide query methods for reporting and analytics.

## Files to Create
- packages/llm-gateway/src/cost-tracker.ts
- packages/llm-gateway/src/pricing-tables.ts
- db/migrations/XXX_create_llm_costs.sql
- packages/llm-gateway/test/cost-tracker.spec.ts

## Dependencies
Task 014-001, ADR-007 (Database)