# Observability Infra Guidebook

This folder is the operator and maintainer guidebook for `observability-infra`.

Use these documents in this order:

1. [system-guidebook.md](/home/jainam/repository/observability-infra/docs/system-guidebook.md)
   Start here to understand the full system, why it is shaped this way, and how logs, metrics, and traces move.

2. [component-deep-dive.md](/home/jainam/repository/observability-infra/docs/component-deep-dive.md)
   Read this when you want to understand each component, its config, its responsibilities, and its failure modes.

3. [repository-structure-guide.md](/home/jainam/repository/observability-infra/docs/repository-structure-guide.md)
   Read this when you want to understand how the repository is organized and where to change things.

The existing docs in this folder remain useful for narrower tasks:

- [architecture-overview.md](/home/jainam/repository/observability-infra/docs/architecture-overview.md)
- [application-integration-contract.md](/home/jainam/repository/observability-infra/docs/application-integration-contract.md)
- [deployment-validation-runbook.md](/home/jainam/repository/observability-infra/docs/deployment-validation-runbook.md)
- [generic-rollout-checklist.md](/home/jainam/repository/observability-infra/docs/generic-rollout-checklist.md)
- [grafana-dashboard-spec.md](/home/jainam/repository/observability-infra/docs/grafana-dashboard-spec.md)

## What This Guidebook Covers

- why the stack has both a local and a central collector
- what each component does and does not do
- why specific ports are exposed
- how logs, metrics, and traces flow through the system
- how Grafana is wired to Loki, Tempo, and Prometheus
- how labels and resource attributes are promoted and used
- how the repository is structured
- where to change the system safely

## What It Does Not Cover

- application-side Go instrumentation source code in `go-observability`
- service business logic
- EC2 provisioning or Terraform-style infrastructure automation

