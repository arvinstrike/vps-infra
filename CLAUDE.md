# CLAUDE.md — vps-infra

## What this repo is

Infrastructure-as-code for my personal VPS (DigitalOcean droplet, Ubuntu 24.04, 8GB RAM / 120GB disk).
It hosts the **shared services** that my other projects (`bmn-regulatory-rag`, `praxis-research-agent`) connect to.
This repo is public — it doubles as a portfolio piece demonstrating production ops skills.

## Architecture

- **Caddy** — reverse proxy, automatic HTTPS, one subdomain per service
- **Qdrant** — vector database (used by bmn-regulatory-rag)
- **Postgres 16** — shared relational DB (Langfuse + app databases)
- **Langfuse** (self-hosted) — LLM observability/tracing
- **TEI (text-embeddings-inference, CPU)** — serves `BAAI/bge-m3` embeddings
- **Uptime Kuma** — uptime monitoring
- All services run via a single `docker-compose.yml` on a shared external Docker network named `infra`.
  Project repos join this network; they do NOT define their own copies of shared services.

## Conventions

- Docker Compose v2 syntax (`docker compose`, not `docker-compose`)
- Every service: pinned image tags (no `latest`), `restart: unless-stopped`, memory limits set
- Config as files under version control; secrets ONLY via `.env` (gitignored, `.env.example` kept updated)
- Shell scripts: bash, `set -euo pipefail`, shellcheck-clean
- Commits: Conventional Commits (`feat:`, `fix:`, `docs:`, `chore:`)
- All docs and comments in English

## Hard rules

- NEVER commit secrets, real API keys, IP addresses of the droplet, or SSH details. Use placeholders in docs.
- NEVER expose Qdrant, Postgres, or TEI ports publicly — internal Docker network only. Only Caddy binds 80/443.
- Any new service must be added to `.env.example`, `docs/architecture.md`, and Uptime Kuma.
- Do not disable the firewall (ufw) or fail2ban in any script.

## Common commands

```bash
docker compose up -d              # start/update the stack
docker compose logs -f <service>  # tail logs
docker compose ps                 # status
./scripts/backup.sh               # dump Postgres + Qdrant snapshots
```

## Deploy flow

Push to `main` → GitHub Actions (`.github/workflows/deploy.yml`) → SSH to droplet → `git pull` → `docker compose up -d`.
Deploy secrets live in GitHub Actions secrets, never in the repo.
