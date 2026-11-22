# Learning setup: Free Edition serverless

This repo is a starting point for practising Databricks Declarative Automation Bundles on Databricks Free Edition. The original `flights-simple` example included classic clusters, extra environment targets, and Delta Live Tables pipelines. Those were trimmed so the bundle can deploy and run with serverless only.

## What changed and why

Free Edition does not let you create classic job clusters, and it uses the `workspace` catalog rather than `main`. To get a clean starting point:

- **`databricks.yml`** now has a single `dev` target pointed at `https://dbc-4bb6302f-905a.cloud.databricks.com/`. The `catalog` variable defaults to `workspace`. The `database` variable is the bundle-managed `flights` schema. `include` is `resources/*.yml`, so any job YAML you add under `resources/` is picked up automatically.
- **Classic-cluster jobs and DLT pipelines were removed** from `resources/` (`flights_notebook_job_classic.yml`, `flights_python_script_job_classic.yml`, `pytest_notebook_job_classic.yml`, `notebook_validation_job.yml`, and the `resources/dlt/` folder). They would fail on Free Edition because they start clusters or use features this workspace is not set up for.
- **The existing serverless notebook job was kept.** `resources/flights_notebook_job_serverless.yml` runs `src/flights_main_notebook.py`. Widget defaults in that notebook were updated from `main` to `workspace` / `flights`.
- **Dev mode prefixes schema names.** `targets.dev` uses `mode: development`, so the deployed schema is `workspace.dev_anasbeg_me_flights`, not `workspace.flights`. Use the prefixed name when you query tables in the workspace.

The existing job has already been deployed and run successfully on this workspace. It wrote `workspace.dev_anasbeg_me_flights.flights_raw`.

## How to deploy and run

Use your CLI profile `Anas` on every command (it is not the default profile). From the repo root:

```
databricks bundle validate -t dev -p Anas
databricks bundle deploy -t dev -p Anas
databricks bundle run flights_notebook -t dev -p Anas
```

`dev` is the default target, so `-t dev` is optional, but it is shown here for clarity. After a deploy, you can also open the job in the workspace under **Workflows**.

## Data source

The flights sample data is CSV files at `/databricks-datasets/airlines`. This path is readable on Free Edition serverless.

- There is no header row. The schema is defined in `src/flights/utils/flight_utils.py` (`get_flight_schema()` and `read_batch()`).
- The existing job reads from that path (limited to 1,000 rows in the sample notebook) and appends to the `flights_raw` table in the bundle schema.

You can reuse those helpers in your own notebooks, or write your own `spark.read` if you prefer.

## Your exercise

Leave this branch as the starting point. Do the work on a new branch, then open a pull request back to `main`.

1. Create a branch, for example `feature/ingest-curate-workflow`.
2. Add `src/ingest_flights` as a notebook that reads the flights sample data and writes a raw table.
3. Add `src/curate_flights` as a notebook that reads the raw table, cleans the data, and writes an aggregated curated table.
4. Add `resources/flights_ingest_curate_job.yml` as a two-task job: ingest first, then curate with `depends_on` pointing at the ingest task. Pass catalog and schema as `base_parameters`. Do not add a cluster definition on the job or tasks — omitting it makes the tasks run on serverless.
5. Run `databricks bundle validate -t dev -p Anas`, then deploy, then run your new job, and confirm the tables in the workspace.
6. Commit, push, and open a PR.

### Hints

- Look at `resources/flights_notebook_job_serverless.yml` and `src/flights_main_notebook.py` for the job YAML shape, notebook widget pattern, and how `${var.catalog}` and `${var.database}` are passed as `base_parameters`. Use those same variables so your tables land in the bundle-managed schema.
- Because of dev mode, the schema you query will be `workspace.dev_anasbeg_me_flights`, even though the schema resource is named `flights`.
- `include: resources/*.yml` already picks up a new file in `resources/`. You do not need to edit `databricks.yml` just to add the job.
- For ingest, `/databricks-datasets/airlines` is a large set of CSVs. Consider limiting rows while you are practising, as the existing notebook does.
- For curate, a simple clean (cast types, drop cancelled or null-key rows) plus an aggregate (for example delays or counts by carrier and date) is enough.
- You can put notebooks in `src/` as `.py` Databricks notebooks (starting with `# Databricks notebook source`) like the existing example.

## Leftover files you can ignore

These were left in the repo so you can browse the original example, but they are not part of the Free Edition starting point:

- `_adhoc/` — exploratory scripts. They still hardcode the `main` catalog and old schema names, so they will not run as-is against this workspace.
- `_cicd/` and `.github/workflows/flights_test.yml` — upstream CI examples. They reference deleted targets such as `test_automated` / `prod`, the removed `notebook_validation_job`, and classic clusters.
- `src/flights_main_dlt.py`, `src/flights_main_dlt_sql.sql`, and `tests/integration/` — leftover DLT and validation examples whose bundle resources were removed.

Unit tests under `tests/unit_utils` and `tests/unit_transforms` still cover the helper functions and are optional local practice.
