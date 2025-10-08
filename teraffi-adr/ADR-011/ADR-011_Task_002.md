# Task: Core Node Schema & Constraints (3-4 hours)
**ADR:** ADR-011  
**Date:** 2025-10-01

## Inputs
- Neo4j connection
- Schema definitions from ADR-011

## Outputs
- Node constraints created
- Indexes created
- Validation scripts

## Acceptance Criteria
- [ ] All node types have UNIQUE constraints on id
- [ ] Property indexes created (industry, location, momentum_score, etc.)
- [ ] Composite indexes for common query patterns
- [ ] Schema validation script passes
- [ ] EXPLAIN shows index usage

## Implementation Details
Create constraints and indexes as specified in ADR-011 schema section. Use IF NOT EXISTS to make scripts idempotent. Verify with EXPLAIN on sample queries.

## Files to Create
- db/neo4j/migrations/001_create_constraints.cypher
- db/neo4j/migrations/002_create_indexes.cypher
- scripts/neo4j/validate-schema.ts

## Dependencies
Task 011-001