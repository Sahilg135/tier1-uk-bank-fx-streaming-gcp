# 02 – Architecture Overview

Sanitized portfolio case-study of Tier-1 UK bank FX streaming on GCP.  
Patterns only — no client code/data.

```mermaid
flowchart LR

  subgraph Producers
    A[OMS/EMS] -->|Trades| P1[Pub/Sub: trades]
    B[Pricing Engine] -->|Quotes| P2[Pub/Sub: quotes]
    C[Custodian] -->|Confirms| P3[Pub/Sub: confirms]
  end

  subgraph Boundary["GCP Security Boundary (VPC-SC, CMEK, SA-IAM)"]
    DF[Dataflow (Beam)<br/>validate + dedup<br/>watermarks<br/>joins<br/>DLQ]
    RAW[BigQuery: raw]
    STG[BigQuery: stage]
    MART[BigQuery: mart]
    DQ[DLQ topics]
    CMP[Cloud Composer]
    OBS[Monitoring / Logging]
  end

  %% Ingestion to processing
  P1 --> DF
  P2 --> DF
  P3 --> DF

  %% Processing outputs
  DF --> RAW
  DF --> DQ
  RAW --> STG --> MART

  %% Orchestration
  CMP --> DF
  CMP --> STG
  CMP --> MART

  %% Observability
  DF -. metrics/logs .-> OBS
  MART -. usage/stats .-> OBS

  %% BI
  MART --> BI[Dashboards / Exports]
