# Central Stack Deployment

## Purpose

This directory contains the central observability stack compose scaffold for the pilot deployment.

It is the single-node stack that will run on the observability EC2 host and provide:

- central OpenTelemetry Collector
- Prometheus
- Loki
- Tempo
- Grafana

## Files

- `docker-compose.yml`
- `.env.example`

## Expected Companion Paths

The compose file mounts these repository-managed configuration paths:

- `collector/central/config.yaml`
- `prometheus/prometheus.yml`
- `loki/config.yaml`
- `tempo/config.yaml`
- `grafana/provisioning/`
- `grafana/dashboards/`

Those files and directories are created by later infra tasks. The compose file is defined first so the service topology and mount contract are fixed before backend-specific configuration is added.

The central collector companion env file lives at:

- `collector/central/.env.example`

The central stack environment must also provide:

- `LOKI_PUSH_ENDPOINT`
- `TEMPO_OTLP_ENDPOINT`
- `TEMPO_OTLP_INSECURE`

## Exposed Ports

- `4317` OTLP/gRPC receiver on the central collector
- `4318` OTLP/HTTP receiver on the central collector
- `13133` collector health endpoint
- `8889` Prometheus scrape endpoint exposed by the central collector
- `9090` Prometheus
- `3100` Loki
- `3200` Tempo
- `3300` Grafana host access mapped to container port `3000`

## Notes

- this stack is intentionally single-node for the pilot
- volume-backed persistence is defined for Prometheus, Loki, Tempo, and Grafana
- Tempo is pinned to `grafana/tempo:2.9.1` in `.env.example` because the current `2.10.x` default single-binary startup path enables Kafka-backed ingest modules that break this simple local-storage pilot stack
- service image tags should be pinned through `.env` before production deployment
