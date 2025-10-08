# Task: Cost Monitoring & Optimization (2-3 hours)
**ADR:** ADR-016  
**Date:** 2025-10-01

## Inputs
- API usage metrics
- LLM token usage
- Database storage metrics

## Outputs
- Cost tracker
- Usage dashboards
- Budget alerts

## Acceptance Criteria
- [ ] Track Twitter API request counts
- [ ] Track Reddit API request counts
- [ ] Track LLM token usage for trend extraction
- [ ] Calculate daily/monthly costs
- [ ] Alert if exceeding budget thresholds
- [ ] Grafana dashboard
- [ ] Optimization recommendations

## Implementation Details
Instrument API calls and LLM requests. Calculate costs based on API pricing. Create dashboard showing usage trends. Set up alerts for budget thresholds.

## Files to Create
- packages/social-listening/src/cost-tracker.ts
- infrastructure/grafana/dashboards/social-listening-costs.json
- infrastructure/prometheus/alerts/social-listening-cost-alerts.yml

## Dependencies
Task 016-010, Monitoring infrastructure