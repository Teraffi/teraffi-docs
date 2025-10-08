# ADR-008: Split Checkout & Fulfillment (Authorize vs Capture)

**Status:** Accepted  
**Date:** 2025-10-02  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-002, ADR-007, ADR-003

---

## Decision
Two workflows: checkout (authorize) and fulfillment (capture at ship). Fraud on-hold state; compensations to release inventory & void auth.

## Implementation Details
Capture failure matrix (transient/permanent/ambiguous); re-auth path; signals/queries; API contracts to start/signal/query.

## Success Metrics
Capture success >99.5%; auth expiry incidents → 0.
