# ADR 0001 — Use GCP native streaming stack

- Status: Accepted
- Date: 2025-11-01

## Context
We need a managed, low-ops streaming pipeline for FX events with exactly-once semantics and warehouse landing.

## Decision
Use **PubSub** (topics: trades/quotes/confirms) → **Dataflow (Beam)** for validate/dedup/watermarks/joins/DLQ → **BigQuery** (raw → stage → mart). Orchestrate maintenance with **Cloud Composer**.

## Consequences
+ Managed scale, backpressure, DLQ handling, SQL-first analytics
− Vendor-specific semantics; must keep labels simple for Mermaid/GitHub
