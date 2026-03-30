# Loki Configuration

## Purpose

This directory contains the canonical Loki configuration for the central observability stack.

Loki is used for:

- log storage
- structured log queries
- trace-to-log correlation in Grafana

This repository uses a single-node filesystem-backed Loki setup for the current phase.

## Files

- `config.yaml`

## Design Notes

- authentication is disabled inside the private observability network
- storage is filesystem-backed for the current single-node deployment
- TSDB schema is used for the current Loki storage layout
- this is a production-familiar single-node baseline, not an HA design

## Runtime Contract

The central stack compose file mounts:

- `loki/config.yaml` into `/etc/loki/local-config.yaml`

The central collector pushes logs to:

- `http://loki:3100/loki/api/v1/push`
