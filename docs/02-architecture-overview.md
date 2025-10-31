# 02 – Architecture Overview

Sanitized portfolio case-study of Tier-1 UK bank FX streaming on GCP.  
Patterns only — no client code/data.

```mermaid
flowchart LR

  subgraph Producers
    A[OMS/EMS] -->|Trades| TGT
    B[Pricing Engine] -->|Quotes| QGT
    C[Custodian] -->|Confirms| CGT
  end

  subgraph Boundary["GCP Security Boundary (VPC-SC, CMEK, SA-IAM)"]
    T[Topic: trades]
    Q[Topic: quotes]
    CFM[Topic: confirms]

    DF[Dataflow Beam: validate, dedup, watermarks, joins, DLQ]
    RAW[BigQuery raw]
    STG[BigQuery stage]
    MART[BigQuery mart]
    DQ[DLQ topics]

    CMP[Cloud Composer]
    OBS[Monitoring & Logging]
  end

  A --> T
  B --> Q
  C --> CFM

  T --> DF
  Q --> DF
  CFM --> DF

  DF --> RAW
  DF --> DQ
  RAW --> STG --> MART

  CMP --> DF
  CMP --> STG
  CMP --> MART

  DF -.-> OBS
  MART -.-> OBS

  MART --> BI[Dashboards / Exports]
