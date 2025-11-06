# SLOs & Observability

**Service Level Objectives**

- E2E latency p95 **< 90 s**
- Delivery success **≥ 99.5 %**
- DLQ rate **< 0.5 %** of daily volume
- Throughput **~6–7 M events/day (~250–400 events/sec)**

**Dashboards & Alerts**

- Dataflow: backlog, watermark skew, worker utilization, error counts  
- BigQuery: slot utilization, query latency, bytes-scanned anomalies  
- Logs-based metrics → alerting (email / on-call)
