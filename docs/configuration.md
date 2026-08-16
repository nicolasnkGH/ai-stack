# Configuration

This guide documents sanitized configuration patterns for the inference-first stack.

## 1) Core endpoint configuration

Use placeholders in public docs and examples.

```env
# Gateway
OPENAI_BASE_URL=https://ai-api.example.com/v1
LITELLM_MASTER_KEY=<REPLACE_ME>

# Backend targets (examples)
LITELLM_PRIMARY_URL=http://10.0.0.10:8081
LITELLM_SECONDARY_FAST_URL=http://10.0.0.20:8082
LITELLM_SECONDARY_BALANCED_URL=http://10.0.0.20:8083
```

## 2) Alias routing intent

Map stable client aliases to backend services so callers do not depend on host/port details.

| Alias | Target URL (example) |
| --- | --- |
| `llama-36b` | `http://10.0.0.10:8081` |
| `llama-9b` | `http://10.0.0.20:8083` |
| `llama-4b` | `http://10.0.0.20:8082` |

## 3) Optional services in this repository

The repository also includes optional UI/auxiliary components under `config/docker-compose.yml` (for example Open WebUI, Redis, SearXNG, TTS, and legacy ComfyUI integration). They are not required for the core routing concept.

## 4) Hermes configuration shape (high level)

```env
HERMES_DB_PATH=/opt/hermes/data/tasks.db
HERMES_WEBHOOK_SECRET=<REPLACE_ME>
TELEGRAM_BOT_TOKEN=<REPLACE_ME>
TELEGRAM_APPROVER_CHAT_ID=<REPLACE_ME>
```

Use approval channels for human-in-the-loop controls before execution.

## 5) Public sanitization checklist

- Domains: use `example.com`.
- IPs: use RFC1918 examples (`10.x.x.x`, `192.168.x.x`).
- Secrets: use `<REPLACE_ME>` placeholders only.
- Paths: avoid uniquely identifying hostnames or usernames.

## Related docs

- [README](../README.md)
- [Architecture](./architecture.md)
- [Installation](./installation.md)
- [Usage](./usage.md)
