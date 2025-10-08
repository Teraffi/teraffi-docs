# ADR-010: Time-Series Partitioning & Archival (Events/Webhooks/Outbox)

**Status:** Accepted  
**Date:** 2025-10-02  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-003, ADR-007, ADR-006

---

## Decision
Monthly partitions; export >13m to Azure Blob (Parquet); drop after 18m; orders keep archived_at.

## Implementation Details
Partitioned DDL; archival exporter with verification; retention matrix; restore runbook with RPO/RTO.

## Success Metrics
Stable query plans; monthly maintenance <5m; restore drills pass.
