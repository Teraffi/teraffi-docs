# Task: Database Exporters (PostgreSQL, Redis, Neo4j) (5-6 hours)
**ADR:** ADR-024  
**Date:** 2025-10-01

## Inputs
- Database connections
- Exporter configurations

## Outputs
- PostgreSQL exporter
- Redis exporter
- Neo4j exporter
- Custom query exporters

## Acceptance Criteria
- [ ] Deploy PostgreSQL exporter
- [ ] Deploy Redis exporter
- [ ] Deploy Neo4j exporter (or use built-in metrics)
- [ ] Configure custom queries for business metrics
- [ ] Add scrape configs to Prometheus
- [ ] Test metric collection
- [ ] Documentation

## Implementation Details
Deploy exporters for each database. Configure to expose relevant metrics (connections, query performance, cache hits, etc.). Add custom queries for business metrics.

## Files to Create
- infrastructure/exporters/postgres-exporter.yml
- infrastructure/exporters/redis-exporter.yml
- infrastructure/exporters/neo4j-exporter.yml
- infrastructure/prometheus/scrape-configs/databases.yml

## Dependencies
Task 024-001