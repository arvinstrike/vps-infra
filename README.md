# vps-infra

Infrastructure-as-code for my personal VPS (DigitalOcean, Ubuntu 24.04, 8GB / 120GB).
Hosts the shared services my other projects connect to. Public repo — also a portfolio piece.

## Services

| Service     | Role                                  | Public? |
|-------------|---------------------------------------|---------|
| Caddy       | Reverse proxy, automatic HTTPS        | 80/443  |
| Uptime Kuma | Uptime monitoring                     | via Caddy |

More services (Postgres, Qdrant, TEI, Langfuse) are added in later phases.

## Setup

```bash
cp .env.example .env      # fill in ACME_EMAIL; leave DOMAIN as placeholder until purchased
docker compose up -d      # start the stack
docker compose ps         # status
```

Caddy logs will spam ACME errors until `DOMAIN` resolves to the droplet — that is expected.

## Uptime Kuma (pre-domain access)

Uptime Kuma binds to loopback only (`127.0.0.1:3001`), never exposed to the internet.
Reach it via SSH tunnel from your machine:

```bash
ssh -L 3001:localhost:3001 arvin@<droplet-ip>
# then open http://localhost:3001
```

Once the Caddy route exists, remove the `127.0.0.1:3001:3001` port mapping in `docker-compose.yml`.

## Common commands

```bash
docker compose logs -f <service>   # tail logs
docker compose up -d               # start/update the stack
docker compose ps                  # status
```
