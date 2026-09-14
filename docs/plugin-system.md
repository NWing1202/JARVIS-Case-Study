# Plugin System

## Design goal

Plugins add capabilities without changing the core dialogue contract. A plugin can observe events, inject bounded context, expose tools, or answer direct module commands.

## Lifecycle

A plugin is discovered from a directory manifest and follows this lifecycle:

```text
discover → load → initialize → enable → run → disable → unload
```

The loader records the manifest, imports the declared entry point, applies plugin-specific configuration, and reports readiness. Initialization failures should leave the plugin disabled rather than preventing the whole assistant from starting.

## Manifest contract

A plugin declares at least:

- a stable name;
- version and description;
- entry point;
- optional dependencies;
- default configuration;
- whether it should be enabled automatically.

The public example is available at [../examples/plugin.example.md](../examples/plugin.example.md).

## Event integration

Plugins can subscribe to core events. Common phases include:

- message reception;
- prompt construction;
- provider response;
- plugin completion;
- background maintenance.

Synchronous handlers must return quickly. Long-running work is moved to an asynchronous task or background queue. Unsubscribing is part of unload so disabled plugins do not retain references or continue work.

## Context injection

A plugin may add context to PromptPipeline. Every injection receives:

- a priority;
- a source label;
- sanitized content;
- a clear boundary from system instructions.

Search results, memory records, and other external content are treated as data. They are filtered before injection and never granted authority over system policy.

## Tools

Tool-capable plugins expose a structured tool schema. The core validates arguments, executes the tool through the plugin boundary, and returns a normalized result. Tools are preferable to free-text overrides because they make permissions, inputs, and outputs explicit.

## Status and observability

Each plugin reports a status such as disabled, initializing, ready, or error. The interface can show module state without exposing internal logs. Errors should be non-fatal by default: a failed optional plugin must not terminate the main response.

## Extension checklist

Before adding a plugin:

1. define the smallest useful capability;
2. declare configuration and dependencies;
3. keep I/O off the main event path;
4. sanitize all external content;
5. return structured results;
6. document privacy and side effects;
7. add focused tests for success, failure, and cancellation paths.
