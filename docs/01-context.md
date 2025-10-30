# Context & Goals

**Problem:** Risk & Compliance need near–real‑time visibility into FX transactions for limit monitoring and regulatory reporting.
**Goals:** 
- Deliver **T+0 analytics** with **p95 E2E < 90s**.
- Enforce **data contracts**, versioning, and **auditable lineage**.
- Keep cost predictable with partition/cluster and autoscaling.

**Non‑functional constraints:**
- Strong governance (**VPC‑SC**, **CMEK**, **SA‑IAM**) and PII handling.
- Replay capability for regulator requests within hours.
- Multi‑region ready; graceful backfills.
