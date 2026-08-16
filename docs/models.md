# Models and Routing Roles

This document describes model roles in an agent-centric, inference-first deployment.

## Routing-first model strategy

Clients call stable aliases; LiteLLM maps aliases to concrete backend services.

| Alias | Backend class | Typical role |
| --- | --- | --- |
| `llama-36b` | Larger model endpoint | Deep reasoning, multi-step analysis |
| `llama-9b` | Mid-tier endpoint | Balanced quality/latency workloads |
| `llama-4b` | Small fast endpoint | Tool calls, automation, lightweight tasks |

## Why aliases matter

- Preserve client compatibility while moving models between hosts.
- Enable controlled migrations and A/B routing.
- Keep operational changes inside the platform boundary.

## Capacity planning guidance

- Reserve larger model endpoints for high-value reasoning paths.
- Route routine automations to smaller models.
- Monitor latency and queue depth per alias, then rebalance placement.

## Notes on legacy UI-era model usage

ComfyUI/image-generation and UI-first workflows may still exist as optional legacy paths in this repo, but they are not the core showcase architecture.

## Related docs

- [README](../README.md)
- [Architecture](./architecture.md)
- [Configuration](./configuration.md)
- [Usage](./usage.md)
