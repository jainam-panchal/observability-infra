# Local Collector Deployment

## Purpose

This directory provides the runnable deployment wrapper for the host-local OpenTelemetry Collector.

Use it on each application EC2 host to run:

- one local collector per host
- OTLP receive endpoints for application containers
- Docker `json-file` log ingestion from the host
- OTLP forwarding to the central collector

The local collector behavior itself is defined in:

- `../../collector/local/config.yaml`

This deployment wrapper is responsible only for how that collector runs on the host.

## Files

- `docker-compose.yml`
- `.env.example`

## Required Host Properties

- Docker installed and running
- Docker log driver set to `json-file`
- `/var/lib/docker/containers` available on the host
- network path from the host to the central collector OTLP endpoint

## Setup

Create a local `.env` file from:

- `.env.example`

Set:

- `CENTRAL_OTLP_ENDPOINT`
- `CENTRAL_OTLP_INSECURE`
- `DEPLOYMENT_ENVIRONMENT`
- `HOST_NAME`
- `HOST_ID`

## Validate Before Startup

Run:

```bash
docker compose -f deploy/local/docker-compose.yml --env-file deploy/local/.env config
```

Pass criteria:

- Compose resolves without syntax errors
- the collector image and env vars resolve correctly

For repository-level definition validation, this also works with:

```bash
docker compose -f deploy/local/docker-compose.yml --env-file deploy/local/.env.example config
```

## Start

Run:

```bash
docker compose -f deploy/local/docker-compose.yml --env-file deploy/local/.env up -d
```

## Validate After Startup

Run:

```bash
docker compose -f deploy/local/docker-compose.yml ps
```

Check health:

```bash
curl -fsS http://localhost:13133/
```

Pass criteria:

- the collector container is running
- the collector health endpoint responds
- application containers can reach `host.docker.internal:4317` in bridge-network deployments or `localhost:4317` in host-network deployments

## Notes

- This wrapper uses published host ports rather than host network mode.
- Application containers in separate Compose stacks should typically send OTLP to `host.docker.internal:4317` when running with standard Docker bridge networking.
- If a host uses `network_mode: host` for the application stack, the app may instead use `localhost:4317`.
