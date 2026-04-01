# Application Integration Contract

## 1. Purpose

This document defines the platform-side integration contract for a generic containerized Go Gin application using the observability stack.

It complements the package-side integration guide in `go-observability`. The package guide explains what application code must do. This document explains what the platform expects from the application and what the platform guarantees in return.

This contract is intentionally generic. It must not assume a single service repository, one project structure, or one deployment pipeline.

## 2. Supported Application Shape

The first-class supported application shape is:

- Go application
- Gin HTTP server for API traffic
- optional worker or background job execution path
- structured logging
- OpenTelemetry instrumentation
- optional GORM-backed database access
- optional outbound HTTP client calls
- containerized deployment on Docker

Applications outside that shape may still be supportable later, but this is the baseline contract for the first implementation.

## 3. Platform Expectations From the Application

The application must:

- send traces and metrics to a local OpenTelemetry Collector
- write structured JSON logs to `stdout`
- set stable service metadata through environment variables
- preserve `context.Context` through request, DB, and HTTP flows
- normalize route labels for HTTP metrics

If the application uses workers, it must also:

- create root job spans when no incoming trace context exists
- emit stable job identifiers in logs and metrics

If the application uses GORM, it must:

- instrument the existing `*gorm.DB`
- use `WithContext(ctx)` for traced query and transaction flows

## 4. Required Environment Variables

At minimum, the platform expects these variables to exist in the application container:

- `OTEL_SERVICE_NAME`
- `OTEL_SERVICE_VERSION`
- `DEPLOYMENT_ENVIRONMENT`
- `OTEL_SERVICE_ROLE` (`api`, `worker`, or another stable role)
- `OTEL_EXPORTER_OTLP_ENDPOINT`
- `OTEL_TRACE_SAMPLING_RATE`
- `OTEL_TRACES_ENABLED`
- `OTEL_METRICS_ENABLED`
- `LOG_LEVEL`

These values must be stable enough to support Grafana filtering and cross-service queries.

## 5. Collector Connectivity Contract

Applications do not communicate directly with Grafana, Prometheus, Loki, or Tempo.

Applications communicate only with the local collector.

The expected endpoint depends on deployment shape:

### Same Compose Network

Use:

```text
OTEL_EXPORTER_OTLP_ENDPOINT=otel-collector:4317
```

### Separate Host-Local Collector With Docker Bridge Networking

Use:

```text
OTEL_EXPORTER_OTLP_ENDPOINT=host.docker.internal:4317
```

and ensure the container can resolve `host.docker.internal`.

### Host Network Mode

Use:

```text
OTEL_EXPORTER_OTLP_ENDPOINT=localhost:4317
```

This is only valid when the application container shares host networking.

## 6. Logging Contract

The platform assumes application logs are:

- JSON formatted
- written to `stdout`
- emitted with consistent service and environment metadata
- safe for centralized storage

Required logging fields where applicable:

- timestamp
- level
- message
- service
- deployment environment
- `trace_id`
- `span_id`
- worker-specific identifiers such as `job_name` or `job_id`

The platform does not support plaintext-only container logging as a preferred mode.

## 7. Tracing Contract

The platform expects:

- one inbound span per API request
- child spans for outbound HTTP calls
- child spans for DB operations where instrumentation exists
- root spans for worker jobs

Trace correlation works only if the application propagates context correctly and logs preserve `trace_id`.

## 8. Metrics Contract

The platform expects applications to emit low-cardinality, useful metrics.

Minimum API metrics:

- request count
- request duration histogram
- active requests

Minimum worker metrics:

- jobs started
- jobs completed
- jobs failed
- job duration histogram

Recommended runtime metrics:

- process CPU
- process memory
- process uptime

## 9. GORM Contract

GORM is a required first-class path for services that use it.

The platform expects:

- traced GORM queries
- traced GORM transaction flows
- DB spans attached to request or worker traces through `context.Context`

The platform does not require the service to abandon GORM or rewrite repositories. It requires only that the existing `*gorm.DB` be instrumented and used with context propagation.

## 10. Container and Docker Contract

For Docker-based applications, the platform expects:

- Docker log driver is `json-file`
- application logs are present in container `stdout`
- local collector has host access to Docker container logs
- application container can reach the local collector

If the deployment uses a separate host-local collector, the application compose stack should not embed a duplicate collector service unless that architecture is explicitly chosen.

## 11. Platform Guarantees

If the application satisfies this contract, the platform should provide:

- traces visible in Tempo and Grafana
- logs visible in Loki and Grafana
- metrics visible in Prometheus and Grafana
- trace-to-log correlation in Grafana
- dashboard filtering by service and environment
- baseline alerts based on collected telemetry

## 12. Validation Requirements

Before considering an application integrated, verify:

- one API request is visible as a trace
- logs for that request are searchable by `trace_id`
- request metrics appear in Grafana
- one DB-backed request shows GORM spans if the app uses GORM
- one worker job is visible as a trace if the app has workers
- worker logs and metrics are visible if the app has workers
- service and environment labels are usable in Grafana filters

## 13. Required Smoke Matrix

The platform integration is considered valid only when these smoke checks pass:

- local collector ingest smoke
- central collector routing smoke
- Prometheus scrape and rules smoke
- Grafana datasource and dashboard smoke
- API end-to-end smoke
- worker end-to-end smoke
- trace-to-log correlation smoke
- critical alert pipeline smoke

Each smoke check should define:

- command or runbook procedure
- expected signal path
- pass criteria

Implementation tasks should reference these smoke checks in the checklist `verification` column before moving to `verified` or `done`.

## 14. Common Integration Failures

- application sends OTLP to the wrong endpoint for the container network mode
- logs are only written to local files inside the container
- route labels use raw URLs with identifiers
- `trace_id` is not included in structured logs
- DB operations ignore `context.Context`
- worker jobs do not create root spans
- application emits high-cardinality metrics that damage Prometheus usability
