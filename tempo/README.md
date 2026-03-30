# Tempo Configuration

## Purpose

This directory contains the canonical Tempo configuration for the central observability stack.

Tempo is used for:

- distributed trace storage
- trace search and inspection in Grafana
- trace-to-log pivot workflows

This repository uses a single-node local-storage Tempo setup for the current phase.

## Files

- `config.yaml`

## Design Notes

- OTLP gRPC and HTTP receivers are enabled inside the container
- local storage and WAL are used for the current single-node deployment
- retention is intentionally simple for the pilot baseline
- this is a production-familiar single-node baseline, not an HA design
- this stack is pinned to `grafana/tempo:2.9.1` because the default `2.10.x` single-binary path enables Kafka-backed ingest modules that are incompatible with this simple local-storage deployment

## Runtime Contract

The central stack compose file mounts:

- `tempo/config.yaml` into `/etc/tempo.yaml`

The central collector exports traces to:

- `tempo:4317`
