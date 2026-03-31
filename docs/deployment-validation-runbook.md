# Deployment And Validation Runbook

## 1. Purpose

This runbook defines how to bring up the observability platform and how to validate that the platform is operating correctly.

It is intentionally written for a generic Docker-based environment using:

- one central observability stack
- one local collector per application EC2 host
- one or more generic Go Gin applications sending OTLP to the local collector

This document is the operator-facing procedure for:

- deployment preparation
- central stack startup
- local collector expectations
- smoke testing
- dashboard and alert validation
- common failure isolation

## 2. Scope

This runbook covers:

- central observability stack deployment
- collector and backend health checks
- logs, metrics, and traces validation
- Grafana datasource and dashboard validation
- critical alert validation planning

This runbook does not yet provide:

- an environment-specific automation script for EC2 provisioning
- a fully automated end-to-end test harness

Those are still future work items. The runbook documents the manual validation path until those assets exist.

## Validation Evidence

The current repository validation baseline has been exercised with synthetic non-production telemetry:

- central stack started successfully from `deploy/central/docker-compose.yml`
- local collector behavior was exercised with a one-off local collector container using `collector/local/config.yaml`
- synthetic traces were sent with `telemetrygen traces`
- synthetic metrics were sent with `telemetrygen metrics`
- synthetic OTLP logs were sent with `telemetrygen logs`
- traces were confirmed in Tempo search
- metrics were confirmed in Prometheus
- logs were confirmed in Loki query results
- Grafana datasources and provisioned dashboards were confirmed through the Grafana API

This proves the platform path end to end without depending on one specific application repository.

Application-specific API and worker validation is still tracked separately in the cross-repo rollout checklist items.

## 3. Repository Inputs

Use these files as the canonical inputs during deployment and validation:

- central stack compose:
  - `deploy/central/docker-compose.yml`
- central stack environment example:
  - `deploy/central/.env.example`
- central collector config:
  - `collector/central/config.yaml`
- local collector config:
  - `collector/local/config.yaml`
- Prometheus config:
  - `prometheus/prometheus.yml`
- Prometheus alert rules:
  - `prometheus/alert-rules.yml`
- Grafana datasources:
  - `grafana/provisioning/datasources/datasources.yml`
- Grafana dashboards:
  - `grafana/dashboards/*.json`

## 4. Preconditions

Before attempting deployment or validation, confirm:

- the central host has Docker and Docker Compose available
- the local application host has Docker available
- the application host uses the Docker `json-file` log driver
- the application containers are configured to write structured JSON logs to `stdout`
- the application containers can reach the local collector
- the local collector can reach the central collector
- required ports are allowed on private networking and security groups

Expected core ports:

- `4317` OTLP gRPC
- `4318` OTLP HTTP
- `3300` Grafana host access mapped to container port `3000`
- `3100` Loki
- `3200` Tempo
- `9090` Prometheus
- `8889` central collector Prometheus exporter
- `13133` collector health endpoint

## 5. Central Stack Deployment

### 5.1 Prepare Environment

Create a central stack environment file from:

- `deploy/central/.env.example`

Populate the backend and collector environment values required by:

- `deploy/central/docker-compose.yml`
- `collector/central/config.yaml`

At minimum, confirm the values used for:

- Loki push endpoint
- Tempo OTLP endpoint
- Tempo TLS behavior
- Grafana admin credentials

Do not hardcode secrets into committed files.

### 5.2 Validate Compose Before Startup

Run:

```bash
docker compose -f deploy/central/docker-compose.yml config
```

Pass criteria:

- Compose resolves without syntax errors
- all referenced files and mounts exist
- service environment expansion succeeds

### 5.3 Start The Central Stack

Run:

```bash
docker compose -f deploy/central/docker-compose.yml up -d
```

Then inspect:

```bash
docker compose -f deploy/central/docker-compose.yml ps
```

Pass criteria:

- `otel-collector` is running
- `prometheus` is running
- `loki` is running
- `tempo` is running
- `grafana` is running

### 5.4 Central Stack Immediate Health Checks

Check collector health:

```bash
curl -fsS http://localhost:13133/
```

Check Prometheus:

```bash
curl -fsS http://localhost:9090/-/ready
```

Check Grafana:

```bash
curl -fsS http://localhost:3300/api/health
```

Pass criteria:

- each endpoint responds successfully
- no container is in a restart loop

## 6. Local Collector Deployment

The local collector deployment wrapper now lives in:

- `deploy/local/docker-compose.yml`

Use it on each application EC2 host together with:

- `deploy/local/.env`
- `collector/local/config.yaml`

Operator expectation:

- run one local collector per application EC2 host
- mount the canonical config from `collector/local/config.yaml`
- mount `/var/lib/docker/containers` read-only into the collector
- expose OTLP on `4317` and optionally `4318`
- provide environment values required by the local collector config
- point the collector at the central collector OTLP endpoint

Minimum local deployment contract:

- OTLP receive enabled for applications
- Docker `json-file` log ingestion enabled
- resource enrichment applied
- batching and memory limiting enabled
- OTLP export to central collector configured

Validate the deployment wrapper before startup:

```bash
docker compose -f deploy/local/docker-compose.yml --env-file deploy/local/.env config
```

For repository definition validation without a host-specific `.env`, this may be checked with:

```bash
docker compose -f deploy/local/docker-compose.yml --env-file deploy/local/.env.example config
```

Then start it:

```bash
docker compose -f deploy/local/docker-compose.yml --env-file deploy/local/.env up -d
```

## 7. Generic Application Validation Contract

Before using any app as a smoke source, verify that the application meets the integration contract from:

- `docs/application-integration-contract.md`

At minimum confirm:

- `OTEL_SERVICE_NAME` is set
- `DEPLOYMENT_ENVIRONMENT` is set
- `OTEL_EXPORTER_OTLP_ENDPOINT` points to the local collector
- traces are enabled
- metrics are enabled
- logs are JSON to `stdout`
- `trace_id` is present in logs when a trace exists

For worker-enabled services, also confirm:

- job spans are emitted
- worker metrics are emitted
- worker logs contain stable job identifiers

## 8. Validation Sequence

Run validation in this order:

1. central stack health
2. collector scrape and routing health
3. Grafana datasource health
4. dashboard definition and loading checks
5. API telemetry path
6. worker telemetry path
7. trace-to-log correlation
8. alert pipeline tests

Do not skip directly to application validation before central stack health is confirmed.

## 9. Smoke Procedures

### 9.1 Collector Config Validation

If the collector binary is available locally, validate config files before startup:

```bash
otelcol-contrib --config collector/central/config.yaml --dry-run
```

```bash
otelcol-contrib --config collector/local/config.yaml --dry-run
```

Pass criteria:

- both configs load without schema or pipeline errors

### 9.2 Prometheus Config Validation

Run:

```bash
promtool check config prometheus/prometheus.yml
```

```bash
promtool check rules prometheus/alert-rules.yml
```

Pass criteria:

- config parses successfully
- rules parse successfully

### 9.3 Prometheus Target Validation

In Prometheus, inspect target health for:

- `central-otel-collector`
- `prometheus`

Validation options:

- Prometheus UI targets page
- Prometheus API

Pass criteria:

- both targets are `up`
- no persistent scrape errors are present

### 9.4 Grafana Datasource Validation

Open Grafana and confirm the provisioned datasources:

- Prometheus
- Loki
- Tempo

Pass criteria:

- all datasources exist without manual UI setup
- Grafana reports each datasource as healthy or queryable

### 9.5 Dashboard Validation

Open the core dashboards:

- Service Fleet
- Service Debug
- Worker Fleet
- Worker Debug
- Observability Platform Health
- Logs Investigation

Pass criteria:

- dashboards load successfully
- variables render
- no panel returns a persistent schema error
- panels query the expected datasource

Data presence is validated later by API and worker smoke tests.

### 9.6 API Telemetry Smoke

Use one generic Go Gin API service as the validation source.

Generate at least one traced request through the app.

Validate:

- a request trace appears in Tempo
- request logs appear in Loki
- request metrics appear in Prometheus and Grafana
- the Service Fleet and Service Debug dashboards filter correctly for the service
- if the API uses GORM, a DB child span appears in the trace

Pass criteria:

- one request is visible across all three signals
- the trace contains expected application and DB/HTTP child spans where applicable
- logs contain `trace_id`

### 9.7 Worker Telemetry Smoke

Use one generic Go worker service as the validation source.

Run at least one job end to end.

Validate:

- a worker root span appears in Tempo
- worker logs appear in Loki
- worker metrics appear in Prometheus and Grafana
- the Worker Fleet and Worker Debug dashboards filter correctly for the service and job type
- if the worker touches the DB, a DB child span appears in the trace

Pass criteria:

- one worker execution is visible across all three signals
- logs contain `trace_id` plus worker identifiers
- job metrics increment as expected

### 9.8 Trace-To-Log Correlation Smoke

Pick a trace from either the API or worker validation flow.

Validate:

- the trace can pivot to Loki using provisioned Grafana correlation
- the correlated logs are relevant to the selected trace
- searching Loki directly by `trace_id` returns the same records

Pass criteria:

- trace-to-log pivot works from Grafana
- direct Loki trace ID filtering confirms the same log set

### 9.9 Alert Pipeline Smoke

Validate the critical baseline alerts where feasible:

- collector down
- Prometheus down
- elevated API 5xx rate
- elevated API p95 latency

Recommended approach:

- simulate in non-production only
- force a short-lived known condition
- verify alert fires
- verify alert clears after recovery

Pass criteria:

- each tested alert enters firing state
- recovery clears the alert
- alert labels and annotations are useful

## 10. Common Failure Isolation

### No Traces In Tempo

Check:

- application OTLP endpoint
- local collector health and config
- local-to-central network reachability
- central collector trace exporter config

### Logs Missing In Loki

Check:

- application logs are on `stdout`
- Docker uses `json-file`
- local collector has `/var/lib/docker/containers` access
- local collector filelog receiver is active
- central collector logs pipeline exports to Loki successfully

### Metrics Missing In Prometheus

Check:

- app emits OTLP metrics
- local collector forwards metrics
- central collector metrics pipeline exports to Prometheus endpoint
- Prometheus scrapes the central collector target

### Grafana Panels Empty

Check:

- datasource health
- panel query metric names and labels
- service and environment variable filters
- whether data exists for the selected time range

### Trace-To-Log Pivot Fails

Check:

- logs preserve `trace_id`
- Loki datasource derived fields are provisioned
- Tempo traces-to-logs configuration is provisioned
- the log field format matches the configured regex

## 11. Evidence To Record

For non-production validation runs, record:

- validation date and environment
- central stack version or compose revision
- app service names used as telemetry sources
- screenshots or exported evidence of:
  - one API trace
  - one worker trace
  - matching logs in Loki
  - matching dashboard panels
  - alert firing and recovery where tested
- any known gaps or flaky behavior

Record the outcome in:

- `tracking/implementation-checklist.csv`

Do not mark runtime validation tasks complete without recorded evidence.

## 12. Current Known Gaps

- end-to-end runtime validation has not yet been recorded in this repository
- host-level metrics for central EC2 CPU, memory, and disk are not yet wired
- Loki and Tempo internal health panels are deferred until backend self-metrics are explicitly enabled

These are known implementation gaps, not hidden assumptions.
