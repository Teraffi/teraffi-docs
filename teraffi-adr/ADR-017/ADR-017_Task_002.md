# Task: State Machine Implementation (5-7 hours)
**ADR:** ADR-017  
**Date:** 2025-10-01

## Inputs
- State transition definitions
- Permission rules
- Database connection

## Outputs
- State machine service
- Transition validation
- Audit logging

## Acceptance Criteria
- [ ] Define all valid state transitions
- [ ] Role-based permission checking
- [ ] Required data validation per transition
- [ ] Automatic audit log creation
- [ ] Neo4j relationship updates for key stages
- [ ] Notification triggers per transition
- [ ] Rollback on transition failure
- [ ] Unit tests for all transitions

## Implementation Details
Implement state machine with transition rules from ADR-017. Validate permissions and required data before allowing transitions. Log all changes to audit table. Update Neo4j PARTNERS_WITH relationships at signed/active/completed stages.

## Files to Create
- packages/deal-flow/src/state-machine.ts
- packages/deal-flow/src/types/transitions.ts
- packages/deal-flow/test/state-machine.spec.ts

## Dependencies
Task 017-001, ADR-011 (Neo4j)