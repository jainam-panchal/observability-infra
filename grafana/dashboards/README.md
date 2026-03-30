# Grafana Dashboards

## Purpose

This directory contains provisionable Grafana dashboard JSON definitions for the observability stack.

Dashboard JSON files are kept in the repository so that:

- dashboard structure is versioned
- changes are reviewable
- environments stay consistent
- dashboard behavior is not dependent on manual UI editing

## Current Dashboards

- `service-overview.json`
- `worker-overview.json`
- `platform-health.json`
- `logs-and-traces-drilldown.json`

## Dashboard Design Rules

- prefer operator-first layouts over exporter-first layouts
- prefer direct data views over fragile selector-heavy dashboards for the current single-stack/operator workflow
- service, environment, route, level, and job dimensions should appear as data in panels first; only add dashboard variables when they materially improve the operator path
- route and job-name dimensions belong in hotspot panels, not as primary global filters
- logs dashboards must be built around the Loki labels the stack actually indexes today
- if a signal is not reliably available yet, explain that in the dashboard text instead of hiding the limitation behind empty charts
