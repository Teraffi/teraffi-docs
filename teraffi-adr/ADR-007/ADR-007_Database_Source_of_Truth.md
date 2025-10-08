# ADR-007: Database as Source of Truth & Idempotency

**Status:** Accepted  
**Date:** 2025-10-02  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-002, ADR-003, ADR-009

---

## Decision
DB primary with unique constraints; Redis cache acceleration; 409 on in-progress.

## Implementation Details
`idempotency_keys` ledger; unique partial index on orders; algorithm: Redis→DB→backfill; 30–90d retention; cleanup job.

## Security & Ops
Premium Redis with AOF; least-privilege; audit.

## Success Metrics
0 duplicates; POST p95 <120ms.
