# Task: Metabase Deployment & Configuration (3-4 hours)
**ADR:** ADR-018  
**Date:** 2025-10-01

## Inputs
- ClickHouse connection details
- User requirements
- Azure infrastructure

## Outputs
- Metabase deployed
- Database connections configured
- User accounts created

## Acceptance Criteria
- [ ] Metabase deployed on Azure Container Instance
- [ ] ClickHouse database connected
- [ ] User authentication configured (SSO if possible)
- [ ] User roles defined (admin, analyst, viewer)
- [ ] Collections created per persona
- [ ] SSL/TLS enabled
- [ ] Backup strategy for Metabase metadata
- [ ] Documentation

## Implementation Details
Deploy Metabase as container. Connect to ClickHouse warehouse. Configure user authentication and roles. Set up collections for organizing dashboards by persona (executives, creators, operations).

## Files to Create
- infrastructure/metabase/docker-compose.yml
- infrastructure/metabase/config.env
- docs/setup/metabase_setup.md

## Dependencies
Task 018-001, Task 018-005