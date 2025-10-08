# Task: Neo4j Instance Provisioning & Configuration (4-6 hours)
**ADR:** ADR-011  
**Date:** 2025-10-01

## Inputs
- Azure subscription
- Container Instance or VM specs
- Network configuration

## Outputs
- Running Neo4j instance
- Connection credentials
- Basic health checks passing

## Acceptance Criteria
- [ ] Neo4j Community 5.x deployed
- [ ] Accessible via Bolt (port 7687)
- [ ] TLS enabled
- [ ] Application user created with limited privileges
- [ ] Health endpoint responding
- [ ] Connection pooling configured

## Implementation Details
Deploy Neo4j as Azure Container Instance (or VM). Configure bolt+s:// protocol. Create neo4j_app_user with reader/writer roles. Set up connection limits and timeout configurations.

## Files to Create
- infrastructure/neo4j/docker-compose.yml (or Bicep template)
- infrastructure/neo4j/neo4j.conf
- packages/graph/src/connection.ts
- docs/runbooks/neo4j_setup.md

## Dependencies
ADR-001 (Runtime), Azure subscription