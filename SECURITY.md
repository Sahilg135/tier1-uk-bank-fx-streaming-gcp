# Security Model (Patterns)

This case study demonstrates enterprise patterns on Google Cloud:
- **Network:** VPC Service Controls (perimeter-protected BigQuery, GCS, Dataflow, Composer).
- **Encryption:** Customer-Managed Encryption Keys (CMEK) on BigQuery & Cloud Storage.
- **Identity:** Service-account scoped IAM, least privilege, and workload identity for jobs.
- **Data Privacy:** PII masking/obfuscation at the curated layer; redaction UDFs; audit logs retained.
- **Operations:** Logs-based alerts, error budgets, and SLO monitoring for latency & delivery.

> This is not production code. Treat it as reference architecture.
