# ADR-006: Application-Level Tenant Filtering (RLS Backstop Only)

**Status:** Accepted  
**Date:** 2025-10-02  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-001, ADR-003, ADR-007

---

## Decision
Kysely TenantScopeGuardPlugin rejects unscoped queries. RLS on select sensitive tables only.

## Implementation Details
AST inspection requires tenant predicates; JOIN same-tenant rule; SQL lint gate; index policy; EXPLAIN verification.

## Security & Ops
Feature-flag rollout; incident playbook; audit.

## Success Metrics
0 cross-tenant leaks; DB p95 stable.
