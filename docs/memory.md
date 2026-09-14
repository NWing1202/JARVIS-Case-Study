# Memory Architecture

## Purpose

Long-term memory lets J.A.R.V.I.S. retain useful facts about the user, preferences, projects, and recurring context across conversations. Memory is an optional plugin and is kept separate from chat history.

## Pipeline

```text
conversation
    ↓
fact extraction
    ↓
validation and deduplication
    ↓
local SQLite store
    ↓
profile update + full-text index
    ↓
recall for relevant prompts
```

## Storage model

The local store keeps three logical areas:

- **memories**: dated facts, preferences, and episodes;
- **user profile**: normalized attributes such as language, role, location, or preferences;
- **conversation summaries**: compact session-level context for future recall.

A full-text index supports fast recall. Exact and normalized deduplication prevents the same fact from being stored repeatedly. Importance and access counters help rank useful memories.

## Extraction

After a completed exchange, a background extraction step asks a provider for structured facts. The extractor is instructed to return only user-relevant information and to exclude infrastructure details, provider limits, error codes, and configuration facts. Extraction runs asynchronously so it does not block the conversation.

## Recall

When a new message is prepared, the memory plugin:

1. extracts keywords from the current message;
2. searches recent memories and the user profile;
3. removes records marked as instruction attempts or infrastructure noise;
4. injects the relevant context into PromptPipeline with a controlled priority.

The model receives recalled information as context, not as an unconditional command. The system policy remains higher priority.

## Safety

Memory is a trust boundary because old facts can be stale or manipulated. The design therefore uses:

- source metadata for auditability;
- instruction-attempt detection;
- system-fact filtering;
- normalized deduplication;
- bounded recall limits;
- explicit nudge/confirmation for sensitive memory changes where applicable;
- separation between local storage and provider calls.

## Privacy

Memory data remains local to the private deployment. Public documentation does not include real memories, profiles, database files, chat exports, or identifiers. A production deployment should define retention, export, deletion, and access policies appropriate for its users.

See [memory.svg](../assets/memory.svg) for the visual flow.
