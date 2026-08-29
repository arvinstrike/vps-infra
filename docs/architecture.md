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
| TEI         | `ghcr.io/huggingface/text-embeddings-inference:cpu-1.9` | none — internal only | Serves `BAAI/bge-m3` embeddings; consumed by `bmn-regulatory-rag` |
| Langfuse    | `langfuse/langfuse:2.84.1`  | `langfuse.<DOMAIN>` via Caddy | Self-hosted LLM observability (v2); uses the shared Postgres (`langfuse` DB) |

All shared services are now deployed; further work is backups and CI deploy.

## Memory budget

8GB total on the droplet. Current allocation via `deploy.resources.limits`:

| Service     | Limit  |
|-------------|--------|
| Caddy       | 256M   |
| Uptime Kuma | 512M   |
| Postgres    | 1024M  |
| Qdrant      | 1024M  |
| TEI         | 4096M  |
| Langfuse    | 1024M  |
| **Total**   | ~7.75GB |

TEI (CPU embedding inference) is the heaviest service. Its warmup allocation scales
with `--max-batch-tokens` (set to 2048 in compose; the 16384 default OOMs on this box).
Side effect: with `--auto-truncate`, effective max input length is capped to 2048 tokens
— keep RAG chunks under that. Adding Langfuse fills the budget (~7.75GB of 8GB): these are
caps not reservations (real usage is far lower), but any further service needs the TEI cap
trimmed or a bigger droplet.

## Secrets

All credentials live in `.env` (gitignored), templated in `.env.example`.
Nothing sensitive is ever committed — droplet IP, SSH details, and real
credentials are excluded from this repo by hard rule (see `CLAUDE.md`).
