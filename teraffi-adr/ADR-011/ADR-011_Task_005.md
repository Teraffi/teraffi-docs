# Task: Cultural Trend Detection Queries (4-6 hours)
**ADR:** ADR-011  
**Date:** 2025-10-01

## Inputs
- Neo4j with Trend/Demographic nodes
- Brand/Content relationships

## Outputs
- Trend discovery queries
- Trend monitoring service
- API endpoints

## Acceptance Criteria
- [ ] Emerging trend detection (momentum > 0.8, <6 months old)
- [ ] Brand-trend alignment scoring
- [ ] Multi-hop influence analysis
- [ ] Results include evidence and confidence scores
- [ ] Queries optimized (PROFILE shows good plan)

## Implementation Details
Implement cultural trend queries from ADR-011. Calculate relevance scores combining momentum, demographic overlap, and engagement. Filter by brand values and industry.

## Files to Create
- packages/trend-service/src/queries.ts
- packages/trend-service/src/service.ts
- packages/api/src/routes/trends.ts
- packages/trend-service/test/trend-detection.spec.ts

## Dependencies
Task 011-002, Task 011-003 (data must be synced)