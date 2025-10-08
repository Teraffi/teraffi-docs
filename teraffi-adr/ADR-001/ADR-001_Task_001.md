# Task: Initialize Monorepo & CI (pnpm, TS strict, ESLint, Prettier) (4-6 hours)
**ADR:** ADR-001  
**Date:** 2025-10-01

## Inputs
- Node 20
- pnpm
- GitHub Actions

## Outputs
- Workspace files
- CI pipeline
- Lint/typecheck/test scripts

## Acceptance Criteria
- [ ] TS strict enabled
- [ ] CI caches pnpm
- [ ] Lint/typecheck/test pass
- [ ] Formatting enforced

## Implementation Details
- Create workspace, base tsconfig, lint configs; CI job with cache.
- Add root scripts.

## Files to Create
- package.json
- pnpm-workspace.yaml
- tsconfig.base.json
- .github/workflows/ci.yml
- .eslintrc.json
- .prettierrc

## Dependencies
None
