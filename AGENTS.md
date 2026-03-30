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
