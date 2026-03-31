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

Provisioned titles:

- `Service Performance`
- `Background Processing`
- `System Health`
- `Logs And Errors`

## Dashboard Design Rules

- prefer operator-first layouts over exporter-first layouts
- prefer 4 production boards over overlapping split dashboards
- prefer direct data views over fragile selector-heavy dashboards for the current operator workflow
- service and worker dashboards should keep only `environment` and `service` as top-level selectors; route, dependency, and job-name detail belongs inside focused panels and tables
- route and job-name dimensions belong in hotspot panels, not as primary global filters
- use dashboard links for service -> logs and worker -> logs drilldown instead of duplicating selectors across many boards
- logs dashboards must be built around the Loki labels the stack actually indexes today
- if a signal is not reliably available yet, explain that in the dashboard text instead of hiding the limitation behind empty charts
