# Task: Database Schema - Billing Tables (3-4 hours)
**ADR:** ADR-023  
**Date:** 2025-10-01

## Inputs
- Billing requirements
- Stripe data model

## Outputs
- Subscriptions table
- Usage records table
- Invoices table
- Payment attempts table
- Billing events table

## Acceptance Criteria
- [ ] Create subscriptions table with Stripe references
- [ ] Create usage_records table
- [ ] Create invoices table
- [ ] Create payment_attempts table (dunning)
- [ ] Create billing_events table (audit log)
- [ ] All indexes created
- [ ] Foreign key constraints
- [ ] Migration scripts (up/down)
- [ ] Documentation

## Implementation Details
Create PostgreSQL tables for billing system. Include Stripe customer/subscription IDs, usage tracking, invoice details, payment attempt history, and event audit log.

## Files to Create
- db/migrations/XXX_create_subscriptions_table.sql
- db/migrations/XXX_create_usage_records.sql
- db/migrations/XXX_create_invoices.sql
- db/migrations/XXX_create_payment_attempts.sql
- db/migrations/XXX_create_billing_events.sql
- docs/database/billing_schema.md

## Dependencies
ADR-019 (Multi-Tenancy)