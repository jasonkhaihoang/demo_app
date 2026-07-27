# Intent: Sales Data Warehouse

## Goal

Build a comprehensive data warehouse that ingests and transforms Salesforce data into analytics-ready models, enabling sales leadership to analyze pipeline, forecast accuracy, sales performance, and customer metrics.

## Source system

Salesforce CRM

## Target

Fabric Lakehouse (bronze → silver/gold layers)

## Objects in scope

- Salesforce objects (opportunities, accounts, contacts, users, stages, etc. — see Open questions for final table list)
- Analytics models derived from Salesforce data

## Deliverables inventory

Ordered list — one row per deliverable the request names, in request order. Every deliverable noun (source connection, pipeline, mart/model, schedule, semantic model) maps to exactly one row. Gate #1 approves this list.

| # | Deliverable | Kind | Notes |
| --- | --- | --- | --- |
| 1 | Salesforce connection | source connection | Configure dlt connector to Salesforce |
| 2 | Salesforce ingestion pipeline | pipeline | Land Salesforce objects into bronze layer |
| 3 | Sales analytics models | mart/model | Build silver/gold models for key sales metrics (opportunities, forecasts, performance) |

## Success criteria

- Salesforce source connection is verified and test runs successfully
- All required Salesforce tables are ingested into bronze layer with full historical data
- Sales analytics models are built and accessible for reporting
- Data is refreshed on a defined schedule (frequency TBD)

## Out of scope

- BI/reporting dashboards (this delivery covers the data layer only)
- Non-Salesforce data sources
- Custom schema contracts or complex transformations beyond standard medallion architecture

## Open questions

- Which specific Salesforce objects should be ingested? (Opportunity, Account, Contact, User, Stage, Lead, etc. — what's the priority list?)
- What is the desired refresh frequency for the Salesforce data? (daily, weekly, real-time?)
- What is the target grain for key facts? (transaction-level, daily aggregate, etc.)
- Are there specific KPIs or metrics the models must support? (win rate, forecast accuracy, pipeline coverage, etc.?)
- Are there existing Salesforce field customizations we should be aware of?

## Approvals

The coordinator flips this only after a successful `AskUserQuestion` response of `approved`. Do not check by inference. Design, ship, and breaking-schema-delta approvals consolidate in `design.md`.

- [x] User approved intent — `2026-07-27 09:02` (UTC)
