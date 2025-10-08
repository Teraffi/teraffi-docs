# Task: Structured Logging Setup (4-5 hours)
**ADR:** ADR-024  
**Date:** 2025-10-01

## Inputs
- Logging library (Pino)
- Log levels
- Format requirements

## Outputs
- Structured logger
- Log correlation (trace IDs)
- Log levels configuration

## Acceptance Criteria
- [ ] Set up Pino logger
- [ ] Configure JSON formatting
- [ ] Add trace/span IDs to logs
- [ ] Configure log levels per environment
- [ ] Create convenience methods (logError, logAudit, etc.)
- [ ] Standard log fields (service, timestamp, level)
- [ ] Error serialization with stack traces
- [ ] Unit tests

## Implementation Details
Configure Pino for structured JSON logging. Integrate with OpenTelemetry to add trace context. Create helper functions for common log patterns. Configure appropriate log levels.

## Files to Create
- packages/core/src/logging/logger.ts
- packages/core/src/logging/log-formatters.ts
- packages/core/test/logging.spec.ts

## Dependencies
Task 024-003