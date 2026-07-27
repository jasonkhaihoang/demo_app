# Design: Sales Data Warehouse

`design.md` is the single durable plan + evidence ledger for this intent. Reproduce these sections in order.

## Architecture

**Platform**: DuckDB (sandbox) — platform-agnostic medallion dbt models transferable to Fabric/Snowflake/Databricks.

**Grain & Materialization**:
- Bronze (ingestion): Raw Salesforce objects (Opportunity, Account, Contact, User, Stage, Lead) at transaction grain
- Silver (staging): Cleaned, deduplicated Salesforce entities with normalized keys
- Gold (marts): Fact tables (`fct_opportunity_monthly_account`, `fct_sales_metrics`) and dimension tables (`dim_account`, `dim_user`) at daily/account grain

**Technical Approach**:
1. dlt ingestion pipeline with Salesforce verified connector → bronze (raw delta table per resource)
2. dbt staging/intermediate/mart layers → silver/gold
3. DuckDB sandbox for development; models are platform-agnostic and lift to Fabric/Snowflake without change

**Key Decisions**:
- **Verified Salesforce connector**: Use dlt's official Salesforce connector (supports incremental load via SOQL cursors)
- **Medallion architecture**: Bronze → Silver → Gold for data quality and analytics isolation
- **Full historical sync**: Initial load captures all Salesforce history; incremental cursors on subsequent runs
- **Schema contracts**: dlt schema_contract set to `evolve` during discovery, then promoted to `freeze` on production resources to prevent breaking changes
- **Control columns**: Gold-layer facts include `_loaded_at` (pipeline run timestamp) and `_dbt_invocation_id` (dbt invocation identifier) for lineage and debugging

## Pipeline Inventory

Status of ingestion resources (one row per Salesforce object to be ingested). Rows stay at `Status: working` until the artifact exists and evidence is recorded in the Build Plan.

| # | Resource | Status | Entry Point | Schema Contract: Tables | Schema Contract: Columns | Schema Contract: Data Type |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | opportunity | working | `Opportunity` | evolve | evolve | ignore |
| 2 | account | working | `Account` | evolve | evolve | ignore |
| 3 | contact | working | `Contact` | evolve | evolve | ignore |
| 4 | user | working | `User` | evolve | evolve | ignore |
| 5 | stage | working | `Stage` | evolve | evolve | ignore |
| 6 | lead | working | `Lead` | evolve | evolve | ignore |

Note: Resource list and exact schema contracts will be confirmed after source configuration and discovery (Build Plan step `03-salesforce-discovery`). Initial contracts set to `evolve` to permit discovery; production tables will apply `freeze` after schema is pinned.

## Model Inventory

Planned dbt models by medallion layer. Rows stay at `Status: working` until the artifact exists and evidence is recorded in the Build Plan.

| # | Model | Layer | Grain | Status | Notes |
| --- | --- | --- | --- | --- | --- |
| 1 | `stg_opportunity` | Silver | transaction | working | Cleansed opportunities with normalized keys |
| 2 | `stg_account` | Silver | account | working | Cleansed accounts |
| 3 | `stg_user` | Silver | user | working | Cleansed sales users |
| 4 | `int_opportunity_account_join` | Intermediate | opportunity-account | working | Denormalized opportunity-account relationships |
| 5 | `fct_opportunity_monthly_account` | Gold | account-month | working | Monthly pipeline by account (primary fact table) |
| 6 | `dim_account` | Gold | account | working | Account dimensions (name, region, etc.) |
| 7 | `dim_user` | Gold | user | working | Sales rep dimensions |

## Source Mapping / Discovery

**Salesforce Source Status**: Not yet configured. The source connection must be added via `add-or-update-source` (Build Plan step `01-salesforce-connection`) before schema discovery.

**Planned Discovery**:
- After source configuration, `discovering-source-schema` will introspect available Salesforce objects (Opportunity, Account, Contact, User, Stage, Lead, and any custom objects)
- Resource shape, field list, and incremental cursor support will be pinned to schema contracts in the Pipeline Inventory
- Initial load strategy: full historical sync via SOQL, then incremental via `UpdatedDate` cursor

**Expected Salesforce Objects** (from intent open questions):
- `Opportunity` (opportunities, forecast, stage history)
- `Account` (customer accounts, industry, region)
- `Contact` (account contacts, roles)
- `User` (sales reps, territories)
- `Stage` (pipeline stage definitions)
- `Lead` (potential opportunities)

## Change Impact

**Transformation scope**: This is a fresh data product — no existing dbt models are being modified. All 7 planned models (`stg_*`, `int_*`, `fct_*`, `dim_*`) are new.

**Downstream consumers**: None yet. This delivery establishes the analytics foundation; BI dashboards and reports will consume the gold-layer marts in a future iteration (explicitly out of scope per intent).

**Impact rating**: No-impact — greenfield build, no breaking changes to existing models or consumers.

## Build Plan

Ordered build steps — this is the durable plan; there is no `implementation-plan.md`. Status `working` until evidence is recorded.

**Model Mapping** (all 7 models in Model Inventory map to generation steps):
- Step `07-staging-models` generates: `stg_opportunity`, `stg_account`, `stg_user`
- Step `08-intermediate-models` generates: `int_opportunity_account_join`
- Step `09-gold-facts` generates: `fct_opportunity_monthly_account`, `dim_account`, `dim_user`

**Open Questions Resolution Path**:
- Specific Salesforce objects: step `03-salesforce-discovery` confirms available objects with dlt connector
- Refresh frequency: decided during `01-salesforce-connection` setup; default to daily
- Target grain: `fct_opportunity_monthly_account` at account-month grain; adjustable per feedback
- KPIs and metrics: core opportunity/account/user analytics in steps `07`–`09`; semantic-model layer in future
- Field customizations: flow through silver layer via schema_contract

**Steps**:

- `01-salesforce-connection` — phase: Build — goal: Configure Salesforce dlt source connection and test connectivity — skill: `add-or-update-source` — status: working — evidence: [pending]
- `02-source-test-pass` — phase: Build — goal: Verify Salesforce source connection is live — skill: `test-source-connection` — status: working — evidence: [pending]
- `03-salesforce-discovery` — phase: Build — goal: Discover Salesforce object schema via dlt connector — skill: `discovering-source-schema` — status: working — evidence: [pending]
- `04-pipeline-generation` — phase: Build — goal: Generate dlt ingestion pipeline script with schema contracts — skill: `generating-dlt-pipeline` — status: working — evidence: [pending]
- `05-dlt-sandbox-run` — phase: Build — goal: Run dlt pipeline in DuckDB sandbox (full load) — skill: `running-dlt-in-sandbox` — status: working — evidence: [pending]
- `06-bronze-validation` — phase: Build — goal: Validate bronze tables: row counts, schema, null rates — skill: `ingestion-data-testing` — status: working — evidence: [pending]
- `07-staging-models` — phase: Build — goal: Generate dbt staging layer (stg_* models) with sources.yml — skill: `generating-dbt-model` — status: working — evidence: [pending]
- `08-intermediate-models` — phase: Build — goal: Generate dbt intermediate models (int_* denormalizations) — skill: `generating-dbt-model` — status: working — evidence: [pending]
- `09-gold-facts` — phase: Build — goal: Generate dbt fact tables (fct_*) and dimensions (dim_*) — skill: `generating-dbt-model` — status: working — evidence: [pending]
- `10-dbt-sandbox-run` — phase: Build — goal: Run dbt build in DuckDB sandbox (staging → gold) — skill: `running-dbt-in-sandbox` — status: working — evidence: [pending]
- `11-dbt-unit-tests` — phase: Build — goal: Write and run dbt data tests (null checks, relationships, uniqueness) — skill: `dbt-unit-testing` — status: working — evidence: [pending]
- `12-golden-validation` — phase: Verify — goal: Compare gold tables to baseline (if provided) — skill: `validating-against-baseline` — status: working — evidence: [pending]
- `13-dbt-evaluator` — phase: Verify — goal: Run dbt_project_evaluator for schema coverage and naming — skill: `evaluating-dbt-project` — status: working — evidence: [pending]
- `14-dlt-audit` — phase: Verify — goal: Audit dlt pipeline config and resource shape — skill: `evaluating-dlt-pipeline` — status: working — evidence: [pending]
- `15-register-bronze-sources` — phase: Publish — goal: Register dlt bronze tables as dbt sources in sources.yml — skill: `registering-dbt-sources` — status: working — evidence: [pending]
- `16-document-models` — phase: Publish — goal: Document dbt models (column descriptions, grains, assumptions) — skill: `documenting-dbt-models` — status: working — evidence: [pending]
- `17-document-pipeline` — phase: Publish — goal: Document dlt pipeline (resources, cursors, load package behavior) — skill: `documenting-dlt-pipelines` — status: working — evidence: [pending]
- `18-completion-check` — phase: Verify — goal: Verify all Build Plan steps done with evidence + golden replay/evaluators green — skill: `verifying-completion-claims` — status: working — evidence: [pending]

## Gate Ledger

Append-only evidence. Record each deterministic gate result, reviewer verdicts (pasted JSON), and gate-status markers.

- ✅ Intent gate — User approved intent on 2026-07-27 09:02 UTC
- **Design-reviewer verdict (2026-07-27 09:02 UTC)**:
  - First iteration: BLOCK (6 issues identified; resolved by author)
  - Second iteration: APPROVE (all critical issues resolved; mapping explicit; open questions documented)
- ✅ Design gate — User approved design on 2026-07-27 09:02 UTC

## Approvals

The coordinator flips these only after a successful `AskUserQuestion` response of `approved`. Do not check by inference.

- [x] User approved design — `2026-07-27 09:02` (UTC)
- [ ] User approved ship — `YYYY-MM-DD HH:MM` (UTC)
- [ ] User approved breaking schema delta — `YYYY-MM-DD HH:MM` (UTC, if applicable)
