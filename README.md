# observability-infra

## Repository Purpose

`observability-infra` is the infrastructure and operations repository for the centralized observability platform.

It defines how telemetry is collected, routed, stored, visualized, and alerted on across environments.

Planned repository:

- `github.com/jainam-panchal/observability-infra`

## Responsibilities

This repository will own:

- local and central OpenTelemetry Collector configurations
- Docker Compose assets for the central observability stack
- Prometheus configuration and alert rules
- Loki, Tempo, and Grafana configuration
- Grafana datasource provisioning
- Grafana dashboards
- operational documentation and runbooks
- environment-specific deployment guidance

## Non-Responsibilities

This repository will not own:

- reusable Go instrumentation libraries
- service business logic
- application runtime feature code

## Platform Model

The standard deployment model is:

- one local collector per application EC2 host
- one central collector on the observability EC2 host
- central Prometheus, Loki, Tempo, and Grafana services

Application hosts forward telemetry to the central observability stack. Grafana serves as the operator-facing entry point for dashboards, logs, and traces.

## Design Principles

- prefer standard open-source observability patterns
- keep app-to-platform integration simple through collectors
- separate telemetry generation from telemetry storage
- document pilot limitations explicitly
- start with critical alerts and golden-signal dashboards
- support generic Go Gin applications through documented collector and environment contracts

## Expected Areas

- `collector/`
- `prometheus/`
- `loki/`
- `tempo/`
- `grafana/`
- `deploy/`
- `docs/`
- `tracking/`

The central stack compose scaffold lives at:

- `deploy/central/docker-compose.yml`

The high-level architecture overview lives at:

- `docs/architecture-overview.md`

The implementation checklist lives at:

- `tracking/implementation-checklist.csv`

## Dashboard Baseline

The initial Grafana baseline should include:

- Service Overview dashboard
- Worker Overview dashboard
- Observability Platform Health dashboard
- Logs and Traces Drilldown dashboard

The detailed dashboard specification should live in:

- `docs/grafana-dashboard-spec.md`

## Application Integration Contract

The platform documentation should assume a generic containerized Go Gin application, not one specific service.

Application-facing requirements should be documented as a stable contract:

- application sends OTLP to the local collector
- application writes structured JSON logs to `stdout`
- application sets service and environment metadata through environment variables
- application can be instrumented for Gin, GORM, outbound HTTP, and worker/job flows

The infra repository should document the platform side of this contract without embedding service-specific code.

The application integration contract should live in:

- `docs/application-integration-contract.md`

## Success Criteria

- telemetry from pilot Go services reaches Loki, Tempo, and Prometheus
- Grafana provides logs, traces, and metrics in one place
- baseline dashboards and critical alerts are defined
- platform deployment and operational steps are documented
