# Task: Scheduled Reports & Alerts (4-5 hours)
**ADR:** ADR-018  
**Date:** 2025-10-01

## Inputs
- Report requirements
- Email service
- Scheduling infrastructure

## Outputs
- Report generation service
- PDF reports
- Email delivery
- Alert system

## Acceptance Criteria
- [ ] Monthly executive report (PDF)
- [ ] Weekly partnership digest
- [ ] Stalled deal alerts (>30 days)
- [ ] Trend alerts (emerging/declining)
- [ ] Email delivery with attachments
- [ ] In-app notifications
- [ ] Scheduled via cron or Airflow
- [ ] Unsubscribe mechanism

## Implementation Details
Build report generation service. Create PDF templates. Query analytics warehouse for data. Generate charts/visualizations. Email to stakeholders. Set up alerts for actionable events (stalled deals, trend changes).

## Files to Create
- packages/analytics-api/src/reports/scheduled-reports.ts
- packages/analytics-api/src/reports/pdf-generator.ts
- packages/analytics-api/src/alerts/deal-alerts.ts
- airflow/dags/scheduled_reports.py

## Dependencies
Task 018-005, Email service