# ADR-004: AuthN/Z & Publishable Keys (Azure AD B2C + HMAC)

**Status:** Accepted  
**Date:** 2025-10-02  
**Decision Makers:** Platform Architecture Team  
**Related ADRs:** ADR-001, ADR-006, ADR-009

---

## Context
Admin users via SSO; storefront via publishable keys; verify webhooks.

## Decision
Admin: JWT (B2C JWKS). Storefront: HMAC publishable keys (90d rotation, 24h overlap). Webhooks: HMAC + timestamp window.

## Implementation Details
JWT: iss/aud/exp; roles→scopes; route guards.
Publishable key: `pk_live_<tenant>_<keyId>` + Key Vault secret + Redis cache; expiry check.
Webhook: `X-Webhook-Timestamp` + `X-Webhook-Signature`; 5m skew; exponential retries with DLQ.

## Ops
Rotation runbook; compromised-key revocation; JWKS cache single-flight.

## Security
Keys scoped read-only; rate limited; Key Vault for secrets.

## Success Metrics
0 replay incidents; rotation without downtime.
