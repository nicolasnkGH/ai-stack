# Installation

This guide focuses on deploying and operating the stack in an **inference-first** model.

## Prerequisites

- Linux host(s) with Docker + Docker Compose.
- GPU-capable runtime where applicable (NVIDIA/ROCm, depending on your model placement).
- A sanitized `.env` based on local requirements.

## 1) Prepare environment

```bash
cp .env.example .env
# Fill with local values only (never commit secrets)
```

## 2) Build and deploy workflow

```bash
# Pull registry-managed images
docker compose pull

# Build locally customized services
docker compose build

# Start or refresh deployment
docker compose up -d --remove-orphans
```

## 3) Validate deployment

```bash
docker compose ps
docker compose logs --tail=100
```

If a gateway health endpoint exists in your deployment:

```bash
curl -sSf http://127.0.0.1:4000/health
```

## 4) Update flow

Run this sequence during updates:

1. `docker compose pull`
2. `docker compose build`
3. `docker compose up -d --remove-orphans`
4. Verify with `ps`, logs, and an end-to-end inference request.

## 5) Rollback-first operations

If a release regresses:

- Restore previous image/tag or configuration.
- Restart affected services.
- Re-run validation checks before re-opening traffic.

## 6) Legacy components

This repository still contains legacy/optional UI-first components (including ComfyUI-related paths). Keep them out of critical path unless explicitly needed.

## Related docs

- [README](../README.md)
- [Architecture](./architecture.md)
- [Configuration](./configuration.md)
- [Models](./models.md)
- [Usage](./usage.md)
