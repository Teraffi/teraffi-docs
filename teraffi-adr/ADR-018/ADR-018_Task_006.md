# Task: Airflow DAG Orchestration (4-5 hours)
**ADR:** ADR-018  
**Date:** 2025-10-01

## Inputs
- Extract scripts
- dbt models
- Scheduling requirements

## Outputs
- Airflow DAGs
- Task dependencies
- Error handling and retries

## Acceptance Criteria
- [ ] Main ETL DAG (hourly schedule)
- [ ] Extract tasks (PostgreSQL, Neo4j)
- [ ] dbt run task
- [ ] dbt test task
- [ ] Proper task dependencies
- [ ] Retry logic (2 retries, exponential backoff)
- [ ] Alerting on failure
- [ ] Monitoring and logging

## Implementation Details
Create Airflow DAG orchestrating extract → transform → load pipeline. Configure proper dependencies between tasks. Set up retry logic and alerting. Schedule hourly for near real-time analytics.

## Files to Create
- airflow/dags/teraffi_analytics_etl.py
- airflow/dags/utils/extract_helpers.py
- airflow/config/airflow.cfg

## Dependencies
Tasks 018-003, 018-004, 018-005