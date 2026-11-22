# flights_simple

A small Databricks Declarative Automation Bundle for practising notebook and workflow development on Databricks Free Edition (serverless).

This repo started as the `flights-simple` example from [databricks-dab-examples](https://github.com/databricks-solutions/databricks-dab-examples) and was trimmed so it can deploy and run on Free Edition. See **[LEARNING.md](LEARNING.md)** for what changed, how to deploy and run, and the ingest/curate exercise.

## What's here now

- `databricks.yml` — bundle config with a single `dev` target pointed at a Free Edition workspace, `workspace` catalog, and a bundle-managed `flights` schema.
- `resources/flights_notebook_job_serverless.yml` — a working serverless job that reads the flights sample CSVs and writes `flights_raw`.
- `src/flights_main_notebook.py` — the notebook that job runs.
- `src/flights/` — helper utilities (CSV schema and transforms) you can reuse.

## Quick start

From the repo root, with the Databricks CLI profile `Anas`:

```
databricks bundle validate -t dev -p Anas
databricks bundle deploy -t dev -p Anas
databricks bundle run flights_notebook -t dev -p Anas
```

Full instructions and the learning exercise are in [LEARNING.md](LEARNING.md).
