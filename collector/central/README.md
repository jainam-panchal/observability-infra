# Central Collector Configuration

## Purpose

This directory contains the central OpenTelemetry Collector configuration for the observability EC2 host.

The central collector is the routing and ingestion gateway for telemetry arriving from local collectors on application hosts.

It exists to:

- receive OTLP telemetry from local collectors
- route logs to Loki
- route traces to Tempo
- expose metrics for Prometheus scraping

## Files

- `config.yaml`
- `.env.example`

## Inputs

The collector listens for telemetry on:

- `4317` OTLP/gRPC
- `4318` OTLP/HTTP

## Outputs

The collector exports:

- logs to Loki via push API
- traces to Tempo via OTLP
- metrics via a Prometheus scrape endpoint on `8889`

## Processors

The central collector config currently includes:

- `memory_limiter`
- `batch`

Why they are used:

- `memory_limiter` prevents collector memory exhaustion under burst load
- `batch` reduces exporter overhead and smooths writes to backend systems

## Environment Variables

Required variables:

- `LOKI_PUSH_ENDPOINT`
- `TEMPO_OTLP_ENDPOINT`
- `TEMPO_OTLP_INSECURE`

## Notes

- this file defines central collector behavior only
- Prometheus scrapes the collector rather than receiving direct remote-write pushes from applications
- backend storage configuration for Prometheus, Loki, and Tempo is handled by later infra tasks
