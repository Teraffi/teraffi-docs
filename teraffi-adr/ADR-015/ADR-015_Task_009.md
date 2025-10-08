# Task: Main Affinity Engine Orchestration (6-8 hours)
**ADR:** ADR-015  
**Date:** 2025-10-01

## Inputs
- All component scorers
- Redis cache
- User input and partner entity

## Outputs
- Main AffinityEngine service
- Weighted score calculation
- Recommendation tiers
- Explainability

## Acceptance Criteria
- [ ] Orchestrate all 5 scoring components
- [ ] Multi-level caching (90-day, 30-day, 7-day)
- [ ] Weighted combination (35% semantic, 30% goals, 20% momentum, 10% historical, 5% authenticity)
- [ ] Convert to 0-100 scale
- [ ] Recommendation tiers (strong/good/moderate/weak)
- [ ] Generate explanations (strengths, concerns, timing)
- [ ] Track processing time (20-30s first, 1-2s cached)
- [ ] Integration tests end-to-end

## Implementation Details
Wire together all components. Implement cache-check → triple-extract → populate-graph → score-components → combine → explain flow. Cache at multiple levels for performance.

## Files to Create
- packages/affinity-engine/src/affinity-engine.ts
- packages/affinity-engine/src/cache-manager.ts
- packages/affinity-engine/test/integration.spec.ts
- packages/affinity-engine/test/performance.spec.ts

## Dependencies
Tasks 015-001 through 015-008