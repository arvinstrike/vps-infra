# vps-infra

Infrastructure-as-code for my personal VPS (DigitalOcean, Ubuntu 24.04, 8GB / 120GB).
Hosts the shared services my other projects connect to. Public repo — also a portfolio piece.

## Services

| Service     | Role                                  | Public?   |
|-------------|---------------------------------------|-----------|
| Caddy       | Reverse proxy, automatic HTTPS        | 80/443    |
| Uptime Kuma | Uptime monitoring                     | via Caddy |
| Postgres    | Shared relational DB                   | internal  |
| Qdrant      | Vector DB (embeddings storage)        | internal  |
| TEI         | `bge-m3` embeddings inference (CPU)   | internal  |
| Langfuse    | LLM observability (self-hosted v2)    | via Caddy |

Internal services publish no ports — they are reachable only from the `infra` Docker
network. See `docs/architecture.md` for details.

## Setup

```bash
# 1. External network the stack attaches to (create once; "already exists" is fine)
docker network create infra

# 2. Secrets — copy the template, then generate real values
cp .env.example .env
openssl rand -hex 32      # → paste into POSTGRES_PASSWORD
openssl rand -hex 32      # → paste into QDRANT_API_KEY
# also set ACME_EMAIL; leave DOMAIN as placeholder until purchased

# 3. Start
docker compose up -d      # start/update the whole stack
docker compose ps         # status
```

`.env` is per-machine and gitignored — never committed. Each host (laptop, droplet)
generates its own secrets. Caddy logs will spam ACME errors until `DOMAIN` resolves to
the droplet — that is expected.

## Uptime Kuma

Reachable only through Caddy, at `https://status.<DOMAIN>`. No ports are published
for it directly — it sits on the internal `infra` network only.

## Common commands

```bash
docker compose logs -f <service>   # tail logs
docker compose up -d               # start/update the stack
docker compose ps                  # status
```
