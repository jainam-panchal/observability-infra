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
- `logs-and-traces-drilldown.json`

Provisioned titles:

- `API Debug`
- `Worker Debug`
- `Logs & Errors`

## Dashboard Design Rules

- prefer operator-first layouts over exporter-first layouts
- prefer 3 incident boards over overview-heavy dashboard sprawl
- prefer direct debugging views over fleet-first summaries for the current operator workflow
- service and worker dashboards should keep only `environment` and `service` as top-level selectors; route, dependency, and job-name detail belongs inside focused panels and tables
- route and job-name dimensions belong in hotspot panels, not as primary global filters
- service dashboards should stay HTTP-first and make room for dependency, DB, and logs context in the same page
- use dashboard links for service -> logs and worker -> logs drilldown instead of duplicating selectors across many boards
- logs dashboards must be built around the Loki labels the stack actually indexes today
- remove standalone platform boards unless the stack has true host-level metrics and logs to support them honestly
- if a signal is not reliably available yet, explain that in the panel description instead of hiding the limitation behind empty charts
