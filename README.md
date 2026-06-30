# aetheria-infra

Local + cloud infrastructure for the **Aetheria** project. This repo stands up every backing
service the other Aetheria services depend on, using **Docker Compose**.

> Part of the Aetheria polyrepo (see ADR-0001). Each Aetheria service lives in its own repo and
> runs independently; this repo provides the shared infrastructure they connect to.

## What's inside

| Service | What it is (plain words) | Host port(s) | Web UI |
|---------|--------------------------|--------------|--------|
| **PostgreSQL** | Relational database (tables + SQL) | `5432` | — |
| **Redis** | In-memory cache + presence store | `6379` | — |
| **Kafka** | Durable event log / streaming backbone | `29092` (host), `9092` (containers) | via Kafka UI |
| **Kafka UI** | Dashboard to inspect Kafka topics/messages | `8080` | http://localhost:8080 |
| **RabbitMQ** | Task/command message queue | `5672` (AMQP) | http://localhost:15672 |
| **MinIO** | S3-compatible object storage (files/blobs) | `9000` (API) | http://localhost:9001 |

## Prerequisites
- Docker Desktop (Docker Engine + Compose v2). Check: `docker --version`.

## Quick start
```bash
cp .env.example .env          # create your local config (git-ignored)
docker compose up -d          # start everything in the background (-d = detached)
docker compose ps             # see status + health of each service
docker compose logs -f kafka  # follow one service's logs (Ctrl+C to stop following)
docker compose down           # stop everything (data is kept)
docker compose down -v        # stop AND delete all data (fresh start)
```

## How services connect (important concept)
- **From your machine (host):** use `localhost:<host port>` — e.g. Postgres at `localhost:5432`,
  Kafka at `localhost:29092`.
- **From another container** on the same Compose network: use the **service name** — e.g.
  `postgres:5432`, `kafka:9092`, `redis:6379`. Compose gives each container a DNS name equal to its
  service name. This is why Kafka advertises *two* listeners (one for host, one for containers).

## Default credentials (dev only — change for anything real)
See `.env.example`. User/password default to `aetheria` / `aetheria_dev_pw` across Postgres,
RabbitMQ, and MinIO.

## Notes / future work
- PostgreSQL extensions **pgvector** (vector search for AI memory) and **PostGIS** (geospatial for
  routing) will be added later via a custom image + init scripts (tracked in ROADMAP / an ADR).
- Kubernetes/Helm manifests and Terraform IaC will live here in later phases.
