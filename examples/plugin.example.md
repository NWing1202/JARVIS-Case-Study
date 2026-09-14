# Example plugin contract

This example describes the public contract for an optional J.A.R.V.I.S. plugin. It is documentation, not deployable source code.

## Manifest fields

| Field | Purpose |
|---|---|
| `name` | Stable plugin identifier |
| `version` | Plugin version |
| `description` | Short public description |
| `entry_point` | Module and class loaded by the plugin system |
| `requires` | Optional runtime dependencies |
| `config` | Default values overridden by private configuration |
| `auto_enable` | Whether the plugin starts enabled |

Example manifest shape:

```json
{
  "name": "example",
  "version": "1.0.0",
  "description": "Demonstration plugin with a bounded tool",
  "author": "JARVIS Team",
  "requires": [],
  "auto_enable": false,
  "config": {
    "enabled": false,
    "max_items": 5
  }
}
```

## Lifecycle

1. **Discover**: the loader finds the manifest and records its path.
2. **Load**: the declared class is imported and receives the core reference.
3. **Initialize**: dependencies and subscriptions are prepared.
4. **Enable**: the plugin moves from disabled to ready.
5. **Run**: the core calls commands or tools according to user intent.
6. **Disable/Unload**: subscriptions are removed and resources are closed.

## Event hooks

A plugin may react to a message event and add context during prompt construction. The event payload should be treated as untrusted data. Return quickly from synchronous hooks; delegate slow work to an asynchronous task.

Example event behavior:

```text
MESSAGE_RECEIVED
  → validate input
  → perform bounded preparation
  → publish a non-fatal result

PROMPT_BUILDING
  → add sanitized context with priority 10
  → keep system policy above plugin context
```

## Tool contract

A tool exposes:

- a stable name;
- a human-readable description;
- a JSON schema for arguments;
- a normalized string or structured result;
- explicit error handling.

The core validates arguments before execution. A plugin must not execute arbitrary shell commands or access files outside its declared permission boundary.

## Privacy checklist

- Do not log tokens, keys, raw private messages, or user identifiers.
- Store only the minimum data required for the capability.
- Sanitize external content before prompt injection.
- Document retention and deletion behavior.
- Make optional failures non-fatal to the main conversation.
