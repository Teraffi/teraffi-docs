# Task: Configuration & Secrets Management (2-3 hours)
**ADR:** ADR-014  
**Date:** 2025-10-01

## Inputs
- Azure Key Vault
- Environment configuration
- Provider credentials

## Outputs
- Configuration loader
- Key Vault integration
- Environment-specific configs

## Acceptance Criteria
- [ ] Load API keys from Key Vault
- [ ] Environment-based configuration (dev/staging/prod)
- [ ] Timeout, retry, rate limit configs
- [ ] Cost alert thresholds
- [ ] Validate configuration on startup
- [ ] Hot reload for non-sensitive configs
- [ ] Documentation

## Implementation Details
Load provider API keys from Azure Key Vault. Define configuration schema with validation. Support environment-specific overrides. Provide sensible defaults.

## Files to Create
- packages/llm-gateway/src/config.ts
- packages/llm-gateway/src/config-loader.ts
- packages/llm-gateway/src/config-validator.ts
- config/llm-gateway.yaml

## Dependencies
Azure Key Vault, Task 014-001