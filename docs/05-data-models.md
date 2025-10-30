# Data Models (Curated)

## fx_trade_fact
Key fields:
- `trade_id` (PK), `version`, `event_ts`
- `side`, `symbol`, `qty`, `px`, `notional_usd`
- `trader`, `desk`, `venue`, `status`

## fx_limit_breach
- `breach_ts`, `desk`, `symbol`
- `window_notional_usd`, `limit_usd`, `breach_flag`

## fx_risk_snapshot_1min
- `ts`, `desk`, `symbol`, `notional_usd`, `pnl`

**Storage strategy**
- Partition tables by `trade_date`; cluster by (`desk`, `symbol`).
- Materialized views for common aggregates and BI.
