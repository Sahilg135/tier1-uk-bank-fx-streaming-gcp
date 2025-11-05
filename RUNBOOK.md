# FX Streaming Data Pipeline — **RUNBOOK.md**

> **Scope:** Tier‑1 UK Bank (BARX‑style) FX events → Pub/Sub → Dataflow (Beam) → BigQuery (raw→stage→marts), orchestrated by Cloud Composer. Security with VPC‑SC + CMEK. Documentation is fully **sanitized** (no client data).

---

## What we are doing (context)

Operating a production‑grade, real‑time FX streaming pipeline and keeping it stable, secure, and trustworthy.

## Why we are doing it (logic)

Front‑office and regulatory consumers depend on accurate, low‑latency data. This runbook gives deterministic steps to monitor, troubleshoot, and recover with minimal MTTR.

---

## 0) Resource naming (as implemented)

* **GCP Project:** `tier1-uk-bank-fx-sim`
* **Pub/Sub topics:** `fx.trades`, `fx.quotes`, `fx.confirms`, and `fx.dlq` (dead‑letter)
* **Dataflow job / template:** `fx_stream_pipeline` (Streaming Engine ON)
* **BigQuery datasets:** `fx_raw`, `fx_stage`, `fx_mart`, `fx_ops` (QC/ops)
* **BigQuery tables (examples):** `fx_raw.trades_*`, `fx_stage.validated_trades`, `fx_mart.aggregated_positions`, `fx_ops.qc_results`
* **Composer DAGs:** `fx_ingestion_dag`, `fx_qc_dag`, `fx_replay_dag`, `fx_backfill_dag`
* **KMS key (CMEK):** `projects/<kms-project>/locations/<loc>/keyRings/fx-ring/cryptoKeys/fx-cmek`

> Replace bracketed values if your actual names differ; otherwise use these defaults consistently in README and diagrams.

---

## 1) System Overview

* **Producers:**

  * *trades* from OMS/EMS (BARX ecosystem)
  * *quotes* from pricing engine
  * *confirms* from post‑trade/custodian
* **Transport:** Google **Pub/Sub** topics per event type
* **Processing:** **Dataflow** (Apache Beam) — validate, dedup (windowed Distinct), enrich, route; DLQ on error
* **Storage:** **BigQuery** datasets: `fx_raw`, `fx_stage`, `fx_mart` (partitioned & clustered)
* **Orchestration:** **Cloud Composer** (Airflow) — ingestion checks, QC DAGs, replay/compaction/exports
* **Security:** **VPC‑SC** perimeter; **CMEK** via Cloud KMS; least‑privilege SA‑IAM
* **Observability:** Cloud Monitoring dashboards + Error Reporting + BQ audit views
* **SLO:** **p95 end‑to‑end latency < 90s** (Pub/Sub publish → BQ mart row available)

---

## 2) Environments

* **dev** – rapid iteration; relaxed alerting
* **stg** – near‑prod; full QC; can replay from prod snapshots where allowed
* **prod** – restricted; VPC‑SC enforced; change via PR + approval

> **Change control:** All DAG/Beam changes merge via PR + CODEOWNERS review; versioned releases with tags.

---

## 3) Data Contracts (high level)

* **Envelope:** JSON/Avro with required keys per topic

  * `trades`: `trade_id` (UUID), `symbol`, `side`, `qty`, `price`, `event_time`, `trader_id`, `venue`, `notional_ccy`
  * `quotes`: `quote_id`, `symbol`, `bid`, `ask`, `event_time`, `source`
  * `confirms`: `confirm_id`, `trade_id`, `status`, `event_time`
* **Semantics:** UTC timestamps; ISO currency codes; `event_time` monotonic per key
* **Contracts stored in:** `docs/data_contracts/*.json` (sanitized)

---

## 4) Daily Operations Checklist (Prod)

1. **Composer** → DAGs status:

   * `fx_ingestion_dag` **success** in last 24h
   * `fx_qc_dag` **success**; QC tables updated
2. **Dataflow** → Job `fx_stream_pipeline`:

   * Worker health **green**, backlog **near zero**
3. **BigQuery**:

   * New partitions created for `fx_raw.*`
   * Stage→Mart transformations completed
4. **Monitoring**:

   * p95 latency < **90s** (daily trend OK)
   * DLQ count < **1000**
5. **Incidents:** Review Error Reporting; zero unresolved P1

---

## 5) Monitoring & Alerting

**Dashboards** (Cloud Monitoring):

* *FX‑Latency* — E2E latency p50/p95; Dataflow backlog
* *FX‑DLQ* — DLQ depth per topic; replay throughput
* *FX‑BQ Health* — load errors, query slots utilization

**Key alerts**

* `fx-latency-p95-breach` — p95 > 90s for 10m
* `fx-dlq-depth-high` — DLQ > 1000 messages for 15m
* `fx-bq-load-failed` — load job errors > 0
* `fx-dataflow-job-down` — job not running / workers unhealthy

---

## 6) QC (Quality Control) Jobs (Composer → BigQuery)

* **Row‑count reconciliation:** Pub/Sub msg count vs `fx_raw` inserts (±0.1%)
* **Schema validation:** required fields non‑null; datatypes per contract
* **Business rules:** `trade_date <= event_time`, `qty > 0`, `notional > 0`
* **Latency SLO check:** event ingest → mart arrival < 90s (p95)
* **Anomaly scans:** outlier prices vs rolling window; symbol whitelist

**QC outputs:** `fx_ops.qc_results` (table), daily summary email/Slack (sanitized alias)

---

## 7) Replay, Backfill & Compaction

* **Replay DLQ:** `fx_replay_dag` → re‑validate → write to stage → mark processed
* **Backfill:** run `fx_backfill_dag` with `start_ts`, `end_ts` → idempotent writes
* **Compaction:** periodic merge of micro‑batches into partitioned tables to optimize cost/perf

---

## 8) Failure Scenarios **& Mitigation Steps**

> **Conventions:**
>
> * *Detect* = how we notice; *Diagnose* = what to check; *Mitigate* = what to do now; *Fix* = durable action; *Prevent* = long‑term guardrail.

### A. Pub/Sub backlog spike (producer burst / subscriber slowdown)

* **Detect:** Dataflow backlog ↑; p95 latency > 90s; Pub/Sub *Oldest unacked* ↑
* **Diagnose:** Dataflow worker CPU/mem; autoscaling events; hot keys/skew; publisher rate change
* **Mitigate:**

  1. Temporarily **raise max workers** / enable **Streaming Engine**
  2. **Shard keys** if single hot symbol; widen Beam windows
  3. Throttle upstream publisher (if coordinated)
* **Fix:** Optimize transforms (combine, side inputs); add **partitioning/branching** per topic
* **Prevent:** Sizing SLO tests; autoscaling caps tuned; producer rate contracts

### B. Dataflow job crash / workers unhealthy

* **Detect:** Alert `fx-dataflow-job-down`; Error Reporting exceptions
* **Diagnose:** Job logs; recent deploy; schema drift entering transforms
* **Mitigate:**

  1. **Drain** bad job; **restart** last stable template/version
  2. If schema drift: **bypass to DLQ** and keep pipeline live
* **Fix:** Add null‑safe parsing; feature‑flag for new fields
* **Prevent:** Canary deploys; contract‑first validation

### C. BigQuery load failures (invalid schema / quota)

* **Detect:** Alert `fx-bq-load-failed`; BQ load error logs
* **Diagnose:** Mismatch types; unexpected nulls; partition decorator overflow; slots saturated
* **Mitigate:**

  1. Route offending records → **DLQ** immediately
  2. Retry with **write‑truncate** for stage temp tables when safe
* **Fix:** Align schemas; add **RELAXED** mode only if contract permits
* **Prevent:** Pre‑deploy schema checks; slot reservations for ETL windows

### D. DLQ depth continuously growing

* **Detect:** Alert `fx-dlq-depth-high` sustained
* **Diagnose:** Top error codes by message; recent code push; upstream change
* **Mitigate:**

  1. Hot‑patch parser rules to accept benign changes
  2. Run `fx_replay_dag` for a narrow time window after fix
* **Fix:** Tighten contract + versioning; communicate with producers
* **Prevent:** Schema evolution policy; consumer tests in CI

### E. Late data spike (watermarks behind)

* **Detect:** QC latency breach; watermark lag metrics
* **Diagnose:** Venue outage; clock skew; batch dumps from upstream
* **Mitigate:** Temporarily **increase allowed lateness**; widen windows
* **Fix:** Move to event‑time + idempotent **upserts/merges** in stage
* **Prevent:** Producer SLAs; clock sync monitoring

### F. Duplicate events / over‑counting

* **Detect:** QC reconciliation fails; metric jumps without volume change
* **Diagnose:** At‑least‑once retries; replay overlap
* **Mitigate:** Ensure **Beam Distinct** keyed by `trade_id` within window; use **BQ MERGE** on `trade_id`
* **Fix:** Add `insertId` on writes; strengthen idempotency contract
* **Prevent:** Replay windows guarded; de‑dup dashboards

### G. Schema drift (new/renamed fields)

* **Detect:** Parser exceptions; null inflation in stage
* **Diagnose:** Compare incoming payload vs contract
* **Mitigate:** Route unknown fields to **side‑channel**; keep core flow green
* **Fix:** Versioned contracts; code path for vNext
* **Prevent:** Producer change notice; contract registry checks in CI

### H. CMEK / KMS key issue (permission/rotation)

* **Detect:** Permission denied on writes/reads
* **Diagnose:** KMS key IAM; key state (disabled/rotated)
* **Mitigate:** Use **emergency key version** (break‑glass) per policy
* **Fix:** Rotate and re‑grant minimal roles
* **Prevent:** Key‑health alerts; rotation runbook calendarized

### I. VPC‑SC perimeter violation (exfil attempt)

* **Detect:** Access denied; VPC‑SC violation logs
* **Diagnose:** Source network, SA identity, service endpoint
* **Mitigate:** Run workload **inside** perimeter; use **Private Google Access**
* **Fix:** Update perimeter members; service endpoints
* **Prevent:** Regular perimeter tests; least‑privilege SA maps

### J. Composer DAG failures

* **Detect:** Task red; DAG missed schedule
* **Diagnose:** Airflow logs; connection secrets; dependency timing
* **Mitigate:** Rerun failed task; increase pool slots; clear & backfill
* **Fix:** Add retries/exponential backoff; task‑level SLAs
* **Prevent:** Unit tests for DAGs; pinned provider versions

### K. Cost anomaly (BQ/DF overrun)

* **Detect:** Budget alert; cost dashboard spike
* **Diagnose:** Top queries/jobs by slot & bytes; worker‑hours
* **Mitigate:** Pause non‑critical DAGs; cap autoscaling; prune partitions
* **Fix:** Optimize clustering/partitioning; caching; storage TTLs
* **Prevent:** Budgets + alerts; monthly cost reviews

---

## 9) Standard Operating Procedures (SOP)

* **Deploy new Dataflow template**

  1. Build template → stage in GCS (versioned path)
  2. Update Composer variable `DF_TEMPLATE_URI`
  3. Trigger canary in **stg** → verify QC → promote to **prod**
* **Trigger manual replay**

  * Composer → `fx_replay_dag` → set `topic`, `start_ts`, `end_ts` → run; monitor replay metrics
* **Open a P1 incident**

  * Criteria: sustained p95 > 5m over SLO, DLQ > 10k, job down
  * Create incident ticket; page on‑call; start incident doc

---

## 10) Access & Security

* **Perimeter:** VPC‑SC around BQ, GCS, Pub/Sub, Dataflow, Composer
* **Encryption:** CMEK for BQ datasets, GCS buckets, Pub/Sub topics
* **Identities:** Dedicated SAs per component; no user keys; break‑glass account documented
* **Auditing:** Cloud Audit Logs stored 400 days; access reviews monthly

---

## 11) Escalation & Ownership

| Area                     | Owner / Alias            |
| ------------------------ | ------------------------ |
| Data Engineering On‑Call | de‑oncall@barx‑sim.local |
| Platform/SRE             | sre@barx‑sim.local       |
| Security                 | secops@barx‑sim.local    |

> **RTO goal:** Critical flow restored in **< 30 minutes**; stale data reconciled same day.

---

## 12) Change Management

* PRs with linked ADRs; semantic version tags (e.g., `v1.1.0`)
* Release notes summarizing schema or SLO impacts
* Post‑deploy validation: QC green, latency within SLO for 1h

---

## 13) Appendix

### A. Sample QC SQL (row‑count)

```sql
-- trades: previous 15 minutes
WITH src AS (
  SELECT COUNT(1) AS c FROM fx_raw.trades
  WHERE _PARTITIONTIME >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 15 MINUTE)
), tgt AS (
  SELECT COUNT(1) AS c FROM fx_stage.validated_trades
  WHERE _PARTITIONTIME >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 15 MINUTE)
)
SELECT src.c AS pubsub_estimate, tgt.c AS bq_rows, ABS(src.c - tgt.c) / GREATEST(src.c,1) AS rel_diff;
```

### B. Beam dedup sketch

```python
(
  trades
  | beam.Map(lambda r: (r['trade_id'], r))
  | beam.WindowInto(beam.window.FixedWindows(60))
  | beam.Distinct()
)
```

### C. Paths & Consoles

* Pub/Sub topics: `projects/<proj>/topics/fx.trades|quotes|confirms`
* Dataflow job: `fx_stream_pipeline`
* BQ datasets: `fx_raw`, `fx_stage`, `fx_mart`
* Composer DAGs: `fx_ingestion_dag`, `fx_qc_dag`, `fx_replay_dag`

---

*Last updated: {{2025‑11‑05}} | Document is sanitized — no client data.*
