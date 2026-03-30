# Grafana Provisioning

## Purpose

This directory contains Grafana provisioning assets for the central observability stack.

Grafana provisioning is used so that:

- datasources are created automatically on startup
- environments stay consistent
- manual UI-only configuration does not become the source of truth

## Current Structure

- `datasources/`

Future tasks will add:

- dashboard provisioning
- dashboard JSON definitions

## Datasource Contract

The initial datasources are:

- Prometheus
- Loki
- Tempo

The provisioning must preserve:

- Prometheus as the default datasource
- Loki log search support
- Tempo trace search support
- trace-to-log correlation from Tempo to Loki
- derived trace field extraction from Loki logs
