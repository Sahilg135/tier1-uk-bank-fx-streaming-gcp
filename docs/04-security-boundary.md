# 04 – Security Boundary

```mermaid
flowchart LR
  subgraph Boundary["GCP Security Boundary (VPC-SC, CMEK, SA-IAM)"]
    SA[Service Accounts]
    KMS[Customer Managed Keys]
    VPC[VPC Service Controls]
    Buckets[GCS buckets: restricted]
    BQ[BigQuery datasets: restricted]
    Roles[Least-privilege IAM roles]
  end

  Producers[Producers] -->|ingest| Buckets
  Buckets --> BQ
  SA -. used by .-> Producers
  SA -. used by .-> Dataflow[Dataflow jobs]
  SA -. used by .-> Composer[Cloud Composer]
  KMS --> Buckets
  KMS --> BQ
  VPC --> Buckets
  VPC --> BQ
  Roles --> SA

