# Tier-1 UK Bank — FX Streaming (GCP) · Sanitized Case Study

![CI](https://github.com/Sahilg135/tier1-uk-bank-fx-streaming-gcp/actions/workflows/ci.yml/badge.svg)
[![Release](https://img.shields.io/github/v/release/Sahilg135/tier1-uk-bank-fx-streaming-gcp?display_name=tag)](https://github.com/Sahilg135/tier1-uk-bank-fx-streaming-gcp/releases)
[![Docs](https://img.shields.io/badge/Docs-available-blue)](./docs/)
[![License: MIT](https://img.shields.io/badge/License-MIT-lightgrey)](./LICENSE)

_A sanitized data-engineering case study demonstrating a real-time FX streaming pipeline on GCP._

> **Quick Facts**  
> **Use-case:** real-time FX trades/quotes → enriched → analytics  
> **Stack:** GCP (Pub/Sub → Dataflow/Beam → BigQuery) + Composer; VPC-SC, CMEK  
> **Throughput:** ~2–2.5M events/day; E2E **p95 < 90s**  
> **Patterns-only:** no client code/data; fully sanitized  
> **Ops:** markdown lint + pre-commit; protected `main`; semver releases  
> **SLOs:** success ≥99.5%, p95 lat <90s, DLQ <0.5%  
> **Cost guardrails:** BQ partition/cluster, Dataflow autoscaling, logs-based budgets

---

## Docs Index
- [01 – Context](docs/01-context.md)
- [02 – Architecture Overview](docs/02-architecture-overview.md)
- [03 – Sequence (Streaming)](docs/03-sequence-streaming.md)
- [04 – Security Boundary](docs/04-security-boundary.md)
- [05 – Data Models](docs/05-data-models.md)
- [06 – SLOs & Observability](docs/06-slos-observability.md)
- [07 – Cost Controls](docs/07-cost-controls.md)
- [08 – CI/CD](docs/08-ci-cd.md)
- [09 – License](docs/09-license.md)
- [Release Notes](https://github.com/Sahilg135/tier1-uk-bank-fx-streaming-gcp/releases)

> Note: Sanitized case study from my Cognizant engagement; patterns only—no client code/data.

**TL;DR.** Real-time FX event ingestion, validation, enrichment, and analytics on **GCP** using **Pub/Sub → Dataflow (Apache Beam) → BigQuery**, orchestrated by **Composer**, with **VPC-SC/CMEK** governance. Targets **p95 < 90s** E2E latency at ~**2–2.5M events/day**.  
Security model: see [SECURITY.md](./SECURITY.md).

---

## 📁 Repository Overview
- Real-time ingestion (**Pub/Sub → Dataflow → BigQuery**)
- Orchestration with **Cloud Composer**
- Governance via **VPC-SC & CMEK**
- Observability & **cost-control** docs

## Why this architecture
Risk & compliance require **T+0 visibility** into FX trades. The platform captures trades/quotes, validates & enriches them, and serves curated BigQuery marts and near-real-time dashboards with auditable lineage and low ops overhead.

### Key Outcomes
- **Manual interventions ↓ ~95%** via strong DQ + DLQ flows  
- **Reporting 2× faster**, intra-day risk views  
- **p95 E2E latency < 90s** at peak **350–500 msg/s**  
- **Infra cost ↓ ~35–40%** using BQ partition/cluster + autoscaling

---
## L2 Architecture Overview

```mermaid
flowchart LR
  subgraph Producers
    A["OMS/EMS"] -->|Trades| P1["Pub/Sub: trades"]
    B["Pricing Engine"] -->|Quotes| P2["Pub/Sub: quotes"]
    C["Custodian"] -->|Confirms| P3["Pub/Sub: confirms"]
  end

  subgraph GCP_SecBoundary["GCP Security Boundary (VPC-SC, CMEK, SA-IAM)"]
    DF["Dataflow (Beam)\nvalidate+dedup\nwatermarks/late\nenrich joins\nDLQ routing"]
    BQ["BigQuery\nraw -> stage -> mart\n(partition+cluster)"]
    CMP["Composer\nDAGs: replay, compaction, exports, QC"]
    OBS["Cloud Monitoring & Error Reporting\nSLO p95 < 90s"]

    P1 --> DF
    P2 --> DF
    P3 --> DF
    DF -->|insertId upserts| BQ
    CMP --> BQ
    DF -->|DLQ| DQ["DLQ topics"]
    BQ --> OBS
  end

  BQ --> BI["Power BI / Looker\nDashboards & Alerts"]
```

---

## Event Life-cycle (Trades)

```mermaid
sequenceDiagram
  participant Prod as Producer (OMS/EMS)
  participant PS as Pub/Sub (trades)
  participant DF as Dataflow (Beam)
  participant BQ1 as BigQuery (raw_fx_events)
  participant BQ2 as BigQuery (mart_fx_risk_snapshot)
  participant BI as BI/Exports

  Prod->>PS: publish trade event
  PS->>DF: pull message
  DF->>DF: schema validate, dedup (trade_id+version)
  DF->>DF: enrich from ref data (desk limits, KYC)
  alt valid
    DF->>BQ1: write with insertId (exactly-once)
    BQ1->>BQ2: transform (dbt/SQL, partitions & clusters)
    BQ2-->>BI: dashboards / alerts
  else invalid
    DF-->>PS: route to DLQ with reason
  end
```
---


## Data Model (curated highlights)
- `fx_trade_fact(trade_id PK, version, side, symbol, qty, px, notional_usd, trader, desk, status, event_ts)`
- `fx_limit_breach(desk, symbol, window_notional_usd, limit_usd, breach_flag, breach_ts)`
- `fx_risk_snapshot_1min(ts, desk, symbol, notional_usd, pnl)`

Partition by `trade_date`; cluster by `desk, symbol`. Materialize common aggregates for BI.

---

## SLOs & Observability
- **Delivery success ≥ 99.5%**, **p95 E2E < 90s**, **DLQ rate < 0.5%**
- Cloud Monitoring dashboards on latency, backlog, watermark skew; logs‑based alerts to on‑call.

---

## Author’s responsibilities (what I owned)
- Streaming design on **GCP**; Beam pipelines for **dedup**, **late-data**, **enrichment**, **DLQ**.
- **BigQuery modeling** (raw → stage → mart), partition/cluster strategy, and cost tuning.
- **Security/governance:** **VPC‑SC**, **CMEK**, least‑privilege SA IAM; PII masking UDFs.
- **Ops:** Composer DAGs for replay/compaction/exports; SLOs, alerts, and runbooks.

---

## Repo Map
```
docs/
  01-context.md
  02-architecture-overview.md
  03-sequence-streaming.md
  04-security-boundary.md
  05-data-models.md
  06-slos-observability.md
  07-cost-controls.md
adr/
  0001-record-architecture-decisions.md
ETHICS.md
SECURITY.md
LICENSE
```

> **Sanitization Note:** Public artifacts are generic; client code/data are intentionally excluded. See `ETHICS.md`.

---

## Reuse this pattern

---

### Minimal commit plan (copy the messages)

1. `docs(readme): add quick-facts, real CI+Release badges; link SECURITY; remove duplicate hr`  
2. `docs(readme): fix Repo Map to .md for diagrams; add ADR + Release notes link`  
3. If any of 05–09 pages don’t exist, either **create stubs** or remove links:  
   - `docs: add stubs for data models, slos/observability, cost controls, ci/cd, license`

**If you prefer Option A (rename files):**  
Rename any `docs/*.mmd` → `*.md` and keep the Mermaid code blocks unchanged.

That’s it. Publish `v0.1.1` after these edits and pin the release in LinkedIn Featured.
::contentReference[oaicite:0]{index=0}
