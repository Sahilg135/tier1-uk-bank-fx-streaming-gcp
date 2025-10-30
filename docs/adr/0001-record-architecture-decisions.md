# ADR 0001 — Streaming stack selection (GCP)

- **Context:** Need managed streaming with tight BigQuery integration.
- **Decision:** Use **Pub/Sub + Dataflow (Apache Beam) + BigQuery** with Composer.
- **Status:** Accepted.
- **Consequences:** Exactly-once semantics to BigQuery via insertId; lower ops burden vs. self-managed Kafka/Spark; strong IAM & VPC-SC integration.
