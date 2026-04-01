# Generic Go Service Rollout Checklist

## Purpose

This checklist turns the pilot observability design into a repeatable adoption path for additional Go services.

It is intentionally generic and assumes:

- a containerized Go service
- optional Gin HTTP server
- optional worker execution path
- the `go-observability` package is available
- the platform side from `observability-infra` is already deployed

## 1. Service Readiness

- confirm the service is Go-based
- confirm the service is containerized
- confirm the service writes logs to `stdout`
- confirm Docker uses the `json-file` log driver on the host
- confirm the service can reach the local collector

## 2. Environment Variables

- set `OTEL_SERVICE_NAME`
- set `OTEL_SERVICE_VERSION`
- set `DEPLOYMENT_ENVIRONMENT`
- set `OTEL_SERVICE_ROLE` (`api`/`worker`)
- set `OTEL_EXPORTER_OTLP_ENDPOINT`
- set `OTEL_TRACE_SAMPLING_RATE`
- set `OTEL_TRACES_ENABLED`
- set `OTEL_METRICS_ENABLED`
- set `LOG_LEVEL`

## 3. Application Integration

- initialize `go-observability` during startup
- defer telemetry shutdown
- add Gin middleware if the service exposes HTTP APIs
- use the Zap-compatible contextual logger path
- instrument GORM if the service uses it
- instrument `database/sql` only for raw SQL code paths
- instrument outbound HTTP clients
- add worker/job instrumentation if the service runs background jobs

## 4. Logging Requirements

- emit JSON logs only
- include stable service metadata
- include `trace_id` and `span_id` when context exists
- avoid high-cardinality or sensitive fields

## 5. Metrics Requirements

- verify low-cardinality labels only
- for APIs, confirm request count, latency, and active-request metrics
- for workers, confirm started, completed, failed, and duration metrics

## 6. Trace Requirements

- confirm inbound API spans if the service is HTTP-based
- confirm DB child spans when the service uses GORM or raw SQL helpers
- confirm outbound HTTP child spans where applicable
- confirm root worker spans if the service has jobs

## 7. Platform Validation

- confirm local collector health on `13133`
- confirm central collector health on `13133`
- confirm Prometheus target health
- confirm Grafana datasources are healthy
- confirm the service appears in Tempo
- confirm the service appears in Loki
- confirm the service metrics appear in Prometheus

## 8. Dashboard Validation

- validate the Service Overview dashboard for API services
- validate the Worker Overview dashboard for worker services
- validate the Logs and Traces Drilldown dashboard for the service
- validate the Platform Health dashboard remains healthy after rollout

## 9. Alert Validation

- verify the service participates in baseline error-rate alerts
- verify the service participates in latency alerts where applicable
- verify no unexpected alert noise is introduced

## 10. Completion Criteria

The service rollout is complete only when:

- the service emits traces, metrics, and logs end to end
- Grafana shows the service in the expected dashboards
- trace-to-log correlation works for at least one representative flow
- the rollout does not require business-logic changes
- the validation evidence is recorded in the implementation checklist
