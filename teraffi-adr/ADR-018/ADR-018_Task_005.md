# Task: dbt Project Setup & Core Models (6-8 hours)
**ADR:** ADR-018  
**Date:** 2025-10-01

## Inputs
- Staging data
- Dimensional model design
- Business logic requirements

## Outputs
- dbt project structure
- Staging models
- Intermediate models
- Mart models (dimensions & facts)

## Acceptance Criteria
- [ ] dbt project initialized
- [ ] Staging models (stg_partnerships, stg_entities, etc.)
- [ ] Dimension models (dim_date, dim_entity, dim_trend, dim_deal_stage)
- [ ] Fact models (fact_partnerships, fact_affinity_scores, etc.)
- [ ] Data quality tests (not null, unique, relationships)
- [ ] Documentation
- [ ] Incremental models where appropriate
- [ ] dbt run completes successfully

## Implementation Details
Set up dbt project targeting ClickHouse. Create staging models for raw data. Transform into dimensional models with business logic. Implement data quality tests. Use incremental models for large fact tables.

## Files to Create
- dbt/dbt_project.yml
- dbt/models/staging/stg_partnerships.sql
- dbt/models/staging/stg_entities.sql
- dbt/models/marts/dim_entity.sql
- dbt/models/marts/fact_partnerships.sql
- dbt/models/schema.yml (tests and docs)

## Dependencies
Tasks 018-003, 018-004