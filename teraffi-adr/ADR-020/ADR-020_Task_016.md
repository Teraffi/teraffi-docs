# Task: Rate Limiting & Brute Force Protection (3-4 hours)
**ADR:** ADR-020  
**Date:** 2025-10-01

## Inputs
- Redis for counters
- Security requirements

## Outputs
- Rate limiting middleware
- Brute force detection
- IP blocking

## Acceptance Criteria
- [ ] Rate limit login attempts (5 per minute per IP)
- [ ] Rate limit password reset (3 per hour per email)
- [ ] Account lockout after 5 failed attempts
- [ ] Temporary IP blocks for suspicious activity
- [ ] CAPTCHA after multiple failures (optional)
- [ ] Alert on brute force detection
- [ ] Unit tests

## Implementation Details
Implement rate limiting for auth endpoints. Track failed attempts per IP and per account. Lock accounts temporarily. Consider CAPTCHA integration for high-risk attempts.

## Files to Create
- packages/auth-service/src/middleware/rate-limit.ts
- packages/auth-service/src/brute-force-detector.ts
- packages/auth-service/test/rate-limit.spec.ts

## Dependencies
Task 020-005, ADR-009 (Redis)