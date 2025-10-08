# ADR-005: Feature Flags & Experiments

**Status:** Accepted  
**Date:** 2025-10-02  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-001, ADR-002

---

## Decision
LaunchDarkly primary; Redis fallback evaluator for critical paths.

## Implementation Details
Flag registry; CI drift check; fallback TTL 30s; fail-open for non-security, fail-closed for security-sensitive flags.
Percent rollouts; tenant overrides; dashboards & alerts.

## Ops
Provider outage runbook; progressive rollout templates.

## Success Metrics
No outages due to flag dependency; p95 eval <10ms.
