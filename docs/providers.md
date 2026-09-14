# Provider Strategy

## Abstract contract

JARVIS treats an LLM provider as an adapter with a small common contract:

- initialize a provider instance from configuration;
- send a message or message sequence;
- stream tokens or structured chunks;
- report errors and availability;
- expose provider-specific quota information when available.

The core does not import a concrete provider. It asks Provider Manager for a candidate and consumes the normalized result.

## Candidate chain

A configuration defines one primary provider and an ordered list of fallback candidates. A typical chain is:

```text
primary provider → secondary provider → tertiary provider
```

Candidates are evaluated in order. A candidate can be skipped because credentials are missing, a dependency is unavailable, the model is unsupported, or the provider is still in cooldown.

## Streaming and fallback

Streaming has two important rules:

1. The first usable chunk identifies the provider responsible for the answer.
2. After the first chunk, the implementation does not switch providers for the same user message.

This prevents a response from being assembled from two models. If a provider times out before the first chunk, the manager records the failure and tries the next candidate. If a provider fails after streaming started, the error is surfaced instead of silently mixing output.

## Health and cooldown

Each candidate has a health state:

- `ok`: available for selection;
- `cooling`: recently failed and temporarily deprioritized;
- unavailable: missing configuration or dependency.

Cooldown prevents a failing endpoint from receiving every retry. It also allows a provider to recover without requiring a process restart. The UI can show provider health and the active provider separately, because the configured primary and the provider that actually answered a request may differ.

## Provider families

The public case study intentionally does not list private credentials or exact deployment values. The architecture supports provider families such as:

| Family | Role |
|---|---|
| Groq | Fast primary inference and free-tier model access |
| OpenRouter | Secondary routing and access to a broad model catalog |
| OpenCode Zen | Additional free or low-cost model candidates |
| Gemini / DeepSeek / Kimi / Claude | Optional provider integrations selected by configuration |
| Local / GGUF / Ollama | Offline or locally hosted inference when available |
| Web bridge | Optional compatibility bridge to a local HTTP endpoint |

## Error classification

Provider errors are normalized into user-safe categories: network error, authorization error, rate limit, exhausted credits, unsupported model, timeout, or generic provider error. Detailed diagnostics remain in protected logs.

## Plugin requests

Memory extraction, intent routing, and other background plugin operations use a separate clean request path. These requests do not mutate the active provider or replace the current user response. If all candidates fail, the plugin reports a non-fatal result and the main conversation continues.
