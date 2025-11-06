# Data Contracts v1 (FX Streaming)

## Scope
Topics: `trades`, `quotes`, `confirms`. Timezone: UTC. Units: price=decimal, qty=decimal.

## Common Fields
- `event_id` STRING (UUID) – required, unique per event
- `event_ts` TIMESTAMP UTC – required
- `ingest_ts` TIMESTAMP UTC – system-set
- `source` ENUM {oms, pricing, custodian} – required
- `trace_id` STRING – optional for end-to-end tracing

## trades
- `trade_id` STRING (unique per trade) – required
- `instrument` STRING (e.g., "EUR/USD") – required
- `side` ENUM {BUY, SELL} – required
- `price` NUMERIC(18,6) – required, >0
- `quantity` NUMERIC(18,6) – required, >0
- `account_id` STRING – required

**DQ rules**
- Uniqueness: (`trade_id`) and (`event_id`) must be unique (Dataflow uses `insertId`)
- Late data: accept up to 10 min; else route to DLQ
- Nulls: none allowed in required fields
- Enum guard: reject values outside allowed sets

## quotes
- `quote_id` STRING – required, unique
- `instrument` STRING – required
- `bid` NUMERIC(18,6), `ask` NUMERIC(18,6) – required, `bid < ask`

## confirms
- `confirm_id` STRING – required, unique
- `trade_id` STRING – required (must exist in `trades` within T+1)
- `status` ENUM {PENDING, SETTLED, FAILED} – required

## BigQuery Storage Rules
- Partition: `DATE(event_ts)`
- Cluster: `instrument`, `side` (where applicable)
- Rejections → Pub/Sub DLQ; retries via Composer “replay” DAG.

## Versioning
- Current version: `v1`
- Backward-compatible adds: only new optional fields
- Breaking changes: bump to `v2` and dual-write until cutover
