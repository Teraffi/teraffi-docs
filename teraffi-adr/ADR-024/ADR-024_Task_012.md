# Task: Database Performance Dashboard (4-5 hours)
**ADR:** ADR-024  
**Date:** 2025-10-01

## Inputs
- Database metrics
- Query performance data

## Outputs
- Database dashboard
- Connection pool monitoring
- Query performance tracking

## Acceptance Criteria
- [ ] Connection pool utilization
- [ ] Active/idle connections
- [ ] Query latency (avg, p95, p99)
- [ ] Slow query detection
- [ ] Transaction rate
- [ ] Cache hit rates
- [ ] Disk I/O metrics
- [ ] Table/index sizes
- [ ] Separate panels for PostgreSQL, Redis, Neo4j

## Implementation Details
Create dashboard for database performance. Show connection pools, query performance, cache efficiency. Alert on connection pool exhaustion or slow queries.

## Files to Create
- infrastructure/grafana/dashboards/postgresql.json
- infrastructure/grafana/dashboards/redis.json
- infrastructure/grafana/dashboards/neo4j.json

## Dependencies
Task 024-002, Task 024-009