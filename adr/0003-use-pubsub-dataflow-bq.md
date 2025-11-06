# ADR 0001: Choose Pub/Sub + Dataflow + BigQuery

## Context
We need low-latency FX streaming (quotes, trades, confirms) with managed ops, security (VPC-SC, CMEK), and tight GCP integration.

## Options
1) Kafka on GKE + Flink + self-managed warehouse
2) Pub/Sub + Dataflow (Beam) + BigQuery (managed/serverless)

## Decision
Choose Pub/Sub + Dataflow + BigQuery.

## Consequences
+ Lower ops toil, native CMEK/VPC-SC, quick scale.
- More vendor lock-in vs Kafka/Flink portability.
