# Prometheus Configuration

## Purpose

This directory contains the Prometheus scrape configuration and the initial alert rules for the pilot stack.

Prometheus is responsible for:

- scraping metrics from the central collector
- storing metrics over time
- evaluating alert rules
- serving PromQL queries to Grafana

## Files

- `prometheus.yml`
- `alert-rules.yml`

## Scrape Model

The initial scrape targets are intentionally small:

- `otel-collector:8889`
- `prometheus:9090`

Why:

- the central collector is the canonical metrics exposure point for application telemetry
- Prometheus self-scrape provides a basic health signal for the metrics backend itself

Additional targets such as node exporters can be added later without changing the core stack model.

## Alerting Model

The initial alert set is intentionally narrow:

- central collector down
- Prometheus down
- high 5xx rate for instrumented Go APIs
- high p95 latency for instrumented Go APIs

Why:

- phase one should start with critical alerts only
- these rules match the stated pilot goals without creating early alert noise
- app-level rules assume the `go-observability` HTTP metric names already implemented by the package

## Notes

- Prometheus is scrape-based in this architecture; applications do not push directly to Prometheus
- rules can expand later as dashboard and platform signals mature
