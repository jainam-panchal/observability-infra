# Local Collector Configuration

## Purpose

This directory contains the host-local OpenTelemetry Collector configuration for application EC2 instances.

The local collector is the telemetry agent for one EC2 host. It exists to:

- receive OTLP traces and metrics from local application containers
- ingest Docker `json-file` logs from the host filesystem
- add host and environment metadata
- batch and forward telemetry to the central collector

## Files

- `config.yaml`
- `.env.example`

## Inputs

### OTLP

The collector listens for application telemetry on:

- `4317` OTLP/gRPC
- `4318` OTLP/HTTP

### Docker Logs

The collector expects read-only access to:

- `/var/lib/docker/containers`

This depends on the Docker host using the `json-file` log driver.

## Processors

The local collector config currently includes:

- `memory_limiter`
- `resource`
- `batch`

Why they are used:

- `memory_limiter` prevents the collector from exhausting host memory
- `resource` standardizes host and environment attributes
- `batch` reduces exporter overhead and smooths traffic to the central collector

## Export Path

The collector forwards all three signals to the central collector over OTLP.

Required environment variables:

- `CENTRAL_OTLP_ENDPOINT`
- `CENTRAL_OTLP_INSECURE`
- `DEPLOYMENT_ENVIRONMENT`
- `HOST_NAME`
- `HOST_ID`

## Notes

- this file defines the collector behavior only; deployment wrapper files can be added later
- the local collector remains one-per-host, not one-per-application
- application containers should send OTLP only to this local collector, not directly to Grafana, Loki, Tempo, or Prometheus
