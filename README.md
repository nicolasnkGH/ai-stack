# AI Stack (Inference-First)

Public reference stack for running **local inference endpoints** behind a single **OpenAI-compatible** gateway.

This repo reflects the current direction: agent/API-first usage with `llama.cpp` backends and LiteLLM routing. Legacy UI-heavy/ComfyUI-era workflows are intentionally removed from the primary story.

## Architecture (high-level)

```text
┌──────────────────────┐     ┌────────────────────────────┐
│ AI Agents / Clients  │────▶│ LiteLLM Proxy (:4000)      │
│ (OpenAI-compatible)  │     │ Unified OpenAI endpoint    │
└──────────────────────┘     └──────────────┬─────────────┘
                                             │
                                             ├──▶ alias: llama-36b ──▶ Primary NVIDIA host
                                             │                        http://10.0.0.10:8081
                                             ├──▶ alias: llama-9b  ──▶ Secondary ROCm host (llm-server2)
                                             │                        http://10.0.0.20:8083
                                             └──▶ alias: llama-4b  ──▶ Secondary ROCm host (llm-server2)
                                                                      http://10.0.0.20:8082
```

> LiteLLM keeps one client-facing endpoint while routing model aliases to the correct backend.

## Usage model

- **Inference-first:** Local `llama.cpp` servers provide model endpoints.
- **Gateway-first:** LiteLLM provides unified auth, routing, and OpenAI-compatible API behavior.
- **Agent-first:** Primary consumers are agents, automations, and API clients (with optional UI clients).

Example endpoint shape:

- Public API: `https://ai-api.example.com/v1`
- Internal LiteLLM: `http://127.0.0.1:4000/v1`

## Hermes orchestration (high-level)

Hermes sits above inference and handles operational orchestration:

- Receives events/webhooks from external systems
- Tracks work as tasks on a SQLite-backed kanban board
- Uses Telegram for human-in-the-loop approvals and notifications

This section is intentionally high-level for public safety.

## Quick start

1. Configure environment values in `.env` (use placeholders only in docs):
   ```bash
   cp .env.example .env
   # Example values:
   # LITELLM_MASTER_KEY=<replace-me>
   # LITELLM_PRIMARY_URL=http://10.0.0.10:8081
   # LITELLM_SECONDARY_URL=http://10.0.0.20:8082
   # CF_ACCESS_CLIENT_ID=<replace-me>
   # CF_ACCESS_CLIENT_SECRET=<replace-me>
   ```
2. Start services:
   ```bash
   docker compose pull
   docker compose build
   docker compose up -d
   ```

## Recommended update sequence

```bash
# 1) Pull latest upstream images
docker compose pull

# 2) Rebuild locally customized services
docker compose build --no-cache litellm

# 3) Restart stack
docker compose up -d --remove-orphans
```

> **`pull` vs `build`:** Use `docker compose pull` for services that come from image registries. Use `docker compose build` for services defined by local Dockerfiles/custom config (for example LiteLLM customizations). In mixed stacks, run both.

## Troubleshooting stale builds

If old behavior persists after an update:

```bash
docker compose build --no-cache <service-name>
docker compose rm -sf <service-name>
docker compose up -d <service-name>
docker compose logs -f <service-name>
```
  
