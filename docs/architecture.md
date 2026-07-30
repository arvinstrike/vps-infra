# Architecture

## Network model

One external Docker network, `infra`, shared by every service in this repo and
joined by containers from other project repos (`bmn-regulatory-rag`,
`praxis-research-agent`) that need to reach these shared services.

Only **Caddy** publishes ports on the host (80/443). Every other service has
no `ports:` mapping at all — reachable only from containers on the `infra`
network, never from the public internet.

## Services

| Service     | Image                      | Public route              | Notes |
|-------------|-----------------------------|----------------------------|-------|
| Caddy       | `caddy:2.8-alpine`          | 80/443 (only public bind)  | Reverse proxy, automatic HTTPS via Let's Encrypt |
| Uptime Kuma | `louislam/uptime-kuma:1.23.16` | `status.<DOMAIN>` via Caddy | Uptime monitoring |
| Postgres    | `postgres:16.4-alpine`      | none — internal only       | Shared relational DB; one instance, multiple app databases |
| Qdrant      | `qdrant/qdrant:v1.11.3`     | none — internal only       | Vector DB for `bmn-regulatory-rag`; API-key protected |

Planned next: TEI (bge-m3 embeddings), Langfuse (uses the shared Postgres).

## Memory budget

8GB total on the droplet. Current allocation via `deploy.resources.limits`:

| Service     | Limit  |
|-------------|--------|
| Caddy       | 256M   |
| Uptime Kuma | 512M   |
| Postgres    | 1024M  |
| Qdrant      | 1024M  |
| **Total**   | ~2.75GB |

Leaves >5GB headroom for TEI (CPU embedding inference is the next heaviest
service) and Langfuse, plus room for app containers from other repos.

## Secrets

All credentials live in `.env` (gitignored), templated in `.env.example`.
Nothing sensitive is ever committed — droplet IP, SSH details, and real
credentials are excluded from this repo by hard rule (see `CLAUDE.md`).
