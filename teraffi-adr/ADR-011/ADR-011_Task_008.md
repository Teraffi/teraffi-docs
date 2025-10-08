# Task: Neo4j Backup & Recovery Procedures (2-3 hours)
**ADR:** ADR-011  
**Date:** 2025-10-01

## Inputs
- Neo4j admin tools
- Azure Blob Storage

## Outputs
- Automated backup script
- Restore procedure
- DR documentation

## Acceptance Criteria
- [ ] Daily backups running (cron/scheduled task)
- [ ] Backups uploaded to Azure Blob Storage
- [ ] Retention policy: 7 daily, 4 weekly, 12 monthly
- [ ] Restore procedure tested successfully
- [ ] RTO documented (4 hours), RPO (24 hours)

## Implementation Details
Use neo4j-admin database dump. Schedule via cron (2 AM daily). Upload to geo-redundant Blob Storage. Document restore steps. Test restore quarterly.

## Files to Create
- scripts/neo4j/backup.sh
- scripts/neo4j/restore.sh
- docs/runbooks/neo4j_backup_restore.md
- infrastructure/cron/neo4j-backup.cron

## Dependencies
Task 011-001, Azure Blob Storage configured