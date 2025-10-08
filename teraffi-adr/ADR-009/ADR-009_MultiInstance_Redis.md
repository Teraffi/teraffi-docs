# ADR-009: Multi-Instance Redis Strategy

**Status:** Accepted  
**Date:** 2025-10-02  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-007, ADR-006, ADR-003

---

## Decision
Separate instances: Idempotency (Premium, AOF, noeviction, ≥2 replicas), Read-models (Standard, LRU), Rate-limits (Standard).

## Implementation Details
Client factories; health endpoints; dashboards; AOF lag & memory alerts; chaos drills.

## Success Metrics
Read-model hit >95%; no duplicate effects under failures.
