# Security Policy

This repository is a **sanitized case study** intended for educational and demonstration purposes.  
No production credentials, customer data, or proprietary assets are included.

---

## 🔐 General Guidelines

- **Do not commit any secrets or keys.**  
  Pre-commit checks and GitHub Actions are configured to block accidental secret commits.

- **Use VPC-SC and CMEK for real deployments.**  
  In production, always assume **VPC Service Controls (VPC-SC)** and **Customer-Managed Encryption Keys (CMEK)** across BigQuery, GCS, Dataflow, and Composer.

- **Follow least-privilege IAM.**  
  Assign **service-account–scoped IAM roles** only as required, and avoid using broad `roles/editor` or `roles/owner` privileges.

- **Mask or redact PII.**  
  Personally identifiable data should be anonymized, obfuscated, or masked at ingestion or the curated layer.  
  Use **UDF-based redaction** and audit logs for compliance verification.

- **Enable audit logging.**  
  Retain logs for all security-sensitive actions (dataset changes, job executions, permission grants, and pipeline triggers).

---

## 🧠 Security Model (Patterns)

| Layer        | GCP Services / Controls                                      | Practice Summary |
|---------------|--------------------------------------------------------------|------------------|
| **Network**   | VPC Service Controls (BigQuery, GCS, Dataflow, Composer)     | Perimeter-protected workloads |
| **Encryption**| CMEK on BigQuery and Cloud Storage                           | Customer-managed encryption keys |
| **Identity**  | Service-account IAM, least privilege, workload identity      | Scoped IAM and controlled service boundaries |
| **Privacy**   | Masking, redaction UDFs, audit logs                          | Enforced data minimization and traceability |
| **Operations**| Log-based alerts, SLO monitoring, error budgets              | Latency and delivery reliability guarantees |

---

## 🧾 Responsible Disclosure

If you discover a vulnerability or sensitive data exposure within this repository:

1. **Do not open a public issue.**
2. **Privately report it** via the repository’s “Security” tab → “Report a vulnerability”.
3. Provide clear reproduction steps and context (affected file, branch, commit hash).

All verified reports will be acknowledged and remediated as per standard security response timelines.

---

## ⚠️ Disclaimer

This repository does **not** contain production systems or confidential enterprise data.  
Treat it as a **reference architecture** for demonstrating GCP security patterns.

---

_Last updated: November 2025_
