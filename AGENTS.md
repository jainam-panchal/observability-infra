# AGENTS.md

## Purpose

This repository contains the observability platform and operational assets for:

- local and central OpenTelemetry Collectors
- Prometheus
- Loki
- Tempo
- Grafana
- dashboards, alerts, and runbooks

## Repository Rules

- keep this repository focused on platform and operations concerns
- do not add reusable Go SDK code here
- document platform behavior as a stable contract for generic Go Gin applications
- prefer standard open-source observability patterns over custom conventions

## Required Assets

- collector configs
- Grafana datasource and dashboard definitions
- Prometheus config and alert rules
- architecture and integration docs
- smoke-test and validation runbooks

## Validation Rules

Required smoke areas:

- local collector ingest
- central collector routing
- Prometheus scrape and rules
- Grafana datasource and dashboard health
- end-to-end logs, traces, and metrics flow
- critical alert pipeline

Checklist rule:

- do not mark an infra task `verified` until the referenced smoke row has passed and the result is recorded
- record important evolving implementation context in `AGENTS.md` whenever it will materially affect later work, validation, or integration expectations
- when a checklist verification mapping is too coarse to prove one task honestly, split the smoke row before marking the task complete

## Root Layout Rules

- keep the repo root limited to repository entrypoint files such as `README.md`, `AGENTS.md`, and `.gitignore`
- keep architecture and platform specifications under `docs/`
- keep deployment assets under `deploy/`
- keep execution tracking artifacts under `tracking/`
- the canonical checklist path is `tracking/implementation-checklist.csv`

## Implementation Context

- `go-observability` logger verification now has a dedicated smoke gate `SMK-GO-007`; cross-repo checklist updates should preserve that separation instead of bundling logger proof into broader Gin/http smoke rows
- `go-observability` Gin middleware verification now has a dedicated smoke gate `SMK-GO-008`; cross-repo checklist updates should preserve inbound middleware proof separately from outbound HTTP instrumentation
- `go-observability` outbound HTTP verification remains on `SMK-GO-002`; it now proves `traceparent` injection and client child-span creation independently of inbound Gin middleware behavior
- `go-observability` GORM verification now has a dedicated smoke gate `SMK-GO-009`; preserve that separation instead of bundling it with future raw SQL helper verification
- `go-observability` raw SQL verification remains on `SMK-GO-003`; it proves the secondary `database/sql` path without weakening the package rule that GORM is the primary database integration path
- `go-observability` worker verification remains on `SMK-GO-004`; it proves root job spans, parent propagation, and worker metric emission for the shared helper
- `SMK-DOC-001` is satisfied only when the package guide, platform integration contract, dashboard spec, and implementation checklist agree on env vars, smoke gates, and the current exported integration surface
- example tasks must use dedicated smoke rows that prove the example compiles against the current package surface; do not reuse feature-level smoke rows to verify examples
- `GO-013` must be proved by a testing-specific smoke row that confirms the unit-test inventory is explicit and aligned; do not use the final quality gate row to prove test coverage exists
- `deploy/central/docker-compose.yml` is the canonical central-stack topology file; backend-specific config tasks may add mounted files later, but they should not change the basic service boundary without an explicit architecture decision
- service-facing docs should explain each platform component in plain operational terms: what it is, why it is used, what it is responsible for, and what it is not responsible for
