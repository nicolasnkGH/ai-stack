# Usage Guide

This guide explains day-to-day operation for an inference-first, agent-centric stack.

## 1) Client interaction model

- Agents and services call a single OpenAI-compatible endpoint.
- Request `model` values map to LiteLLM aliases.
- Backend placement remains internal to infrastructure operators.

Example request target:

- `https://ai-api.example.com/v1/chat/completions`

## 2) Operator workflow

1. Confirm service health (`docker compose ps`, recent logs).
2. Validate alias routing behavior with a smoke request.
3. Route heavy tasks to larger aliases, automation to smaller aliases.
4. Track latency/error trends and adjust model placement.

## 3) Hermes orchestration workflow

At a high level:

1. Intake event/webhook.
2. Create/advance kanban task state.
3. Request approval through Telegram.
4. Execute approved task.
5. Report completion/failure status.

This keeps execution auditable and introduces human control points for higher-risk automations.

## 4) Reliability habits

- Prefer incremental updates with quick rollback paths.
- Re-validate after every deployment.
- Keep logs and status checks lightweight but consistent.

## 5) Legacy/optional features

UI and image-generation integrations may be used selectively, but are intentionally de-emphasized in the core operating model.

## Related docs

- [README](../README.md)
- [Architecture](./architecture.md)
- [Configuration](./configuration.md)
- [Installation](./installation.md)
- [Models](./models.md)
