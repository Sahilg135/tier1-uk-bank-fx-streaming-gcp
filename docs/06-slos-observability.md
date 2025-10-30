# SLOs & Observability

**Service Level Objectives**
- E2E latency p95 **< 90s**
- Delivery success **≥ 99.5%**
- DLQ rate **< 0.5%** of daily volume

**Dashboards & Alerts**
- Dataflow: backlog, watermark skew, worker utilization, error counts
- BigQuery: slot utilization, query latency, bytes scanned anomalies
- Logs-based metrics -> Alerting (email/on-call)
