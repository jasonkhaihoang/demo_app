# Intent: Build fct_sales gold model with data reconciliation

## Classification
- **Action:** work
- **Type:** transformation
- **Rationale:** Request to build a dbt gold-layer fact model (fct_sales) with downstream data reconciliation — this is transformation work, not ingestion or advisory.

## Goal
Build a `fct_sales` gold-layer dbt model on DuckDB to serve as the reconciled source of truth for sales metrics, then run data reconciliation to validate the output.

## Source system
Synthetic (fake) data seeded directly into the DuckDB sandbox — no external source connection required.

## Target
DuckDB sandbox (`$VD_EPHM_DUCKDB_PATH`). Gold layer in the DuckDB database, schema `gold`.

## Objects in scope
- `fct_sales` — gold fact table for sales metrics

## Deliverables inventory

| # | Deliverable | Kind | Notes |
| --- | --- | --- | --- |
| 1 | Synthetic bronze source tables | Data seeding | Seed fake orders / transactions data into DuckDB |
| 2 | dbt project scaffolding | Project setup | `dbt_project.yml`, `profiles.yml`, `sources.yml` |
| 3 | Staging models | mart/model | `stg_sales__orders`, `stg_sales__customers` etc. |
| 4 | `fct_sales` gold model | mart/model | Gold fact table for sales metrics |
| 5 | Data reconciliation | validation | Compare gold model against source expectations |

## Success criteria
- `fct_sales` model exists in the DuckDB gold schema with correct grain and measures
- dbt build completes successfully (no errors)
- Data reconciliation passes (row counts, key uniqueness, measure tolerances)

## Out of scope
- External source connections (e.g., Salesforce, PostgreSQL)
- Semantic model / MetricFlow definitions
- Orchestration or scheduling
- Production deployment

## Open questions
- Grain default: **one row per order line item** (pending user confirmation)
- Measures default: `quantity`, `unit_price`, `total_amount`, `discount_amount` (pending user confirmation)
- Dimensions default: `customer`, `product`, `date` (pending user confirmation)

## Approvals

- [ ] User approved intent — `YYYY-MM-DD HH:MM` (UTC)
