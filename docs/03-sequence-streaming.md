# 03 – Sequence (Streaming)

```mermaid
sequenceDiagram
  participant Producer as Producer System
  participant PubSub as PubSub Topic
  participant DF as Dataflow Beam
  participant BQ as BigQuery
  participant CMP as Cloud Composer
  participant BI as Dashboards / Exports

  Producer->>PubSub: publish event
  PubSub->>DF: pull batch
  DF->>DF: validate, dedup, enrich
  DF->>BQ: write to raw
  CMP->>BQ: schedule transform raw->stage->mart
  BQ->>BI: serve reports

