# Cost Controls

- BigQuery **partitioning & clustering**; prune scans with selective predicates.
- **Materialized views** for hot aggregates; BI uses pre-aggregates.
- Dataflow **autoscaling** and **Streaming Engine**; preemptibles for batch.
- Storage lifecycle: raw → NEARLINE after 30 days; export purge policies.
