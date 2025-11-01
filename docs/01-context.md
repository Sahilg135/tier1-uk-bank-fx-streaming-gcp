# 01 – Context

```mermaid
flowchart LR
  subgraph External_Systems
    OMS[OMS/EMS]
    Pricing[Pricing Engine]
    Custodian[Custodian]
  end

  subgraph GCP_Boundary["GCP Security Boundary (VPC-SC, CMEK, SA-IAM)"]
    PubSub[Topics: trades, quotes, confirms]
    Dataflow[Dataflow Beam: validate, dedup, joins, DLQ]
    BQ_R[BigQuery raw]
    BQ_S[BigQuery stage]
    BQ_M[BigQuery mart]
    Composer[Cloud Composer]
    Obs[Monitoring & Logging]
  end

  Consumers[Dashboards / Exports]

  OMS --> PubSub
  Pricing --> PubSub
  Custodian --> PubSub

  PubSub --> Dataflow --> BQ_R --> BQ_S --> BQ_M --> Consumers
  Composer --> Dataflow
  Composer --> BQ_S
  Composer --> BQ_M
  Dataflow -.-> Obs
  BQ_M -.-> Obs
