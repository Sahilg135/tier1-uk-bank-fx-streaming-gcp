# Getting Started (patterns-only)

This repo is sanitized (no client code/data). Use this mini flow to **see the patterns**.

## Prereqs
- BigQuery **Sandbox** (free) in your Google account
- Cloud Shell or local `bq` CLI (optional for the load step)

---

## 1) Create a demo dataset + table (DDL)

```sql
CREATE SCHEMA IF NOT EXISTS fx_demo;

CREATE TABLE IF NOT EXISTS fx_demo.events (
  ts   TIMESTAMP,
  pair STRING,
  px   NUMERIC,
  qty  INT64,
  src  STRING
)
PARTITION BY DATE(ts)
CLUSTER BY pair;
