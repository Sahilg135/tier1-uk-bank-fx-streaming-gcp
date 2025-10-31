```markdown
# 02-architecture-overview.md

```mermaid
flowchart LR
  subgraph Producers
    A[OMS/EMS] -->|Trades| P1[(Pub/Sub: trades)]
    B[Pricing Engine] -->|Quotes| P2[(Pub/Sub: quotes)]
    C[Custodian] -->|Confirms| P3[(Pub/Sub: confirms)]
  end

  subgraph Boundary["GCP Security Boundary (VPC‑SC, CMEK, SA‑IAM)"]
    DF[[Dataflow (Beam)\nvalidate+dedup\nwatermarks\nenrich joins\nDLQ]]
    BQ[(BigQuery\nraw → stage → mart)]
    CMP[Composer]
    OBS[Monitoring/Logging]
    P1 --> DF
    P2 --> DF
    P3 --> DF
    DF --> BQ
    CMP --> BQ
    DF --> DQ[(DLQ topics)]
    BQ --> OBS
  end

  BQ --> BI[Dashboards/Exports]

