# 02 – Architecture Overview

Sanitized portfolio case-study of Tier-1 UK bank FX streaming on GCP.  
Patterns only — no client code/data.

```mermaid
flowchart LR

  %% ─────────── External Producers ───────────
  subgraph Producers
    A[OMS/EMS] -->|Trades| TGT
    B[Pricing Engine] -->|Quotes| QGT
    C[Custodian] -->|Confirms| CGT
  end

  %% ─────────── GCP Security Boundary ───────────
  subgraph Boundary["GCP Security Boundary (VPC-SC, CMEK, SA-IAM)"]
    %% Ingestion topics (Pub/Sub)
    T[(Pub/Sub topic: trades)]:::topic
    Q[(Pub/Sub topic: quotes)]:::topic
    CFM[(Pub/Sub topic: confirms)]:::topic

    %% Stream processing
    DF[[Dataflow (Beam)\nvalidate + dedup\nwatermarks\njoins\nDLQ]]:::proc
    DQ[(DLQ topics)]:::topic

    %% Storage layers (BigQuery)
    RAW[(BigQuery: raw)]:::bq
    STG[(BigQuery: stage)]:::bq
    MART[(BigQuery: mart)]:::bq

    %% Orchestration & Ops
    CMP[Cloud Composer]:::svc
    OBS[Monitoring / Logging]:::svc
  end

  %% Edges from producers into boundary
  A -->|Trades| T
  B -->|Quotes| Q
  C -->|Confirms| CFM

  %% Topics to stream processor
  T --> DF
  Q --> DF
  CFM --> DF

  %% Dataflow outputs
  DF --> RAW
  DF --> DQ

  %% Layered warehouse flow
  RAW --> STG --> MART

  %% Orchestration
  CMP --> DF
  CMP --> STG
  CMP --> MART

  %% Observability
  DF -.metrics/logs.-> OBS
  MART -.usage/stats.-> OBS

  %% BI/Exports
  MART --> BI[Dashboards / Exports]

  %% Styles
  classDef topic fill:#eef,stroke:#36c,stroke-width:1px;
  classDef proc fill:#efe,stroke:#3a3,stroke-width:1px;
  classDef bq fill:#ffe,stroke:#c93,stroke-width:1px;
  classDef svc fill:#f4f4f4,stroke:#777,stroke-width:1px;
