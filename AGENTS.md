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
- `deploy/local/docker-compose.yml` is the canonical local collector deployment wrapper; local-host deployment changes should be made there rather than duplicating ad hoc per-host compose definitions elsewhere
- `collector/local/config.yaml` must keep the `health_check` extension enabled because `deploy/local/docker-compose.yml` publishes port `13133` for host-side smoke validation
- `deploy/local/docker-compose.yml` must validate with `docker compose ... --env-file deploy/local/.env.example config`; do not require an undeclared extra `env_file` dependency that breaks repository-level wrapper validation
- `deploy/central/docker-compose.yml` must pass the central collector exporter environment explicitly (`LOKI_PUSH_ENDPOINT`, `TEMPO_OTLP_ENDPOINT`, `TEMPO_OTLP_INSECURE`); do not rely on undocumented host environment leakage
- `deploy/central/docker-compose.yml` must publish collector health on `13133` because the runbook and smoke checks rely on a host-visible readiness probe for the central collector
- `deploy/central/docker-compose.yml` must publish Grafana on host port `3300`; keep docs, runbook commands, and operator guidance aligned to `http://localhost:3300`
- service-facing docs should explain each platform component in plain operational terms: what it is, why it is used, what it is responsible for, and what it is not responsible for
- `collector/local/config.yaml` is the canonical local collector behavior file; later deployment wrappers should mount it rather than duplicating receiver or pipeline logic elsewhere
- `collector/central/config.yaml` is the canonical central collector behavior file; backend tasks should build around its routing contract instead of bypassing it with direct application-to-backend assumptions
- `loki/config.yaml` is the canonical single-node Loki configuration for the current stack; central deployment changes must keep the collector push endpoint and mounted path `/etc/loki/local-config.yaml` aligned with it
- `tempo/config.yaml` is the canonical single-node Tempo configuration for the current stack; central deployment changes must keep the collector OTLP exporter endpoint and mounted path `/etc/tempo.yaml` aligned with it
- the central stack is currently pinned to `grafana/tempo:2.9.1`; do not upgrade Tempo casually because `2.10.x` enables Kafka-backed ingest modules in the default single-binary startup path and breaks this local-storage pilot stack
- `prometheus/prometheus.yml` and `prometheus/alert-rules.yml` are the canonical metrics and alerting entrypoints; early alert rules should stay intentionally narrow and aligned with the metric names emitted by `go-observability`
- `grafana/provisioning/datasources/datasources.yml` is the canonical datasource definition; Grafana datasource setup should remain file-provisioned rather than UI-managed, and correlation settings should live there
- `grafana/provisioning/dashboards/dashboards.yml` is the canonical dashboard provider definition; dashboard JSON files under `grafana/dashboards/` must be file-provisioned into Grafana instead of relying on manual imports
- dashboard JSON files under `grafana/dashboards/` are the canonical dashboard definitions; each dashboard task should produce a standalone JSON file and use its own smoke row rather than relying on the future full-render smoke
- Prometheus must scrape both `otel-collector:8889` for exported application metrics and `otel-collector:8888` for collector self-metrics; the Platform Health dashboard depends on the self-scrape job `central-otel-collector-self`
- `docs/deployment-validation-runbook.md` is the canonical operator runbook for bring-up and validation; runtime smoke rows should align to its procedures rather than inventing ad hoc validation steps elsewhere
- current dashboard metric contract to preserve unless the shared package changes:
  - service dashboard uses `http_server_request_count`, `http_server_request_duration_bucket`, and `http_server_active_requests`
  - worker dashboard uses `worker_job_started_total`, `worker_job_completed_total`, and `worker_job_duration_bucket`
  - platform dashboard uses `up`, `otelcol_exporter_sent_*`, `otelcol_exporter_send_failed_*`, `otelcol_exporter_queue_size`, `otelcol_processor_dropped_*`, `process_resident_memory_bytes`, and `process_cpu_time_seconds_total`
  - logs and traces drilldown dashboard depends on Loki log labels and fields carrying `service_name`, `deployment_environment`, `level`, and `trace_id`, plus the provisioned Tempo and Loki correlation settings
  - current dashboard label keys are `service_name`, `deployment_environment`, `instance`, `http_route`, `http_response_status_code`, `job_name`, and `job_status`
- if exported metric names or label keys change in `go-observability`, update this file and the affected dashboard JSON definitions in the same change set
- current local deployment standard: use `deploy/local/docker-compose.yml` with the canonical `collector/local/config.yaml`, host Docker log mounts, and host-published OTLP ports unless an explicit host-network requirement is documented
- current central runtime baseline: central compose now depends on repository-owned `loki/config.yaml` and `tempo/config.yaml`; if backend ports, mounted paths, or collector exporter targets change, update all three in the same change set
