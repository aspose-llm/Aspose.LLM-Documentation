---
weight: 35
date: "2026-04-23"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/cache-management/
feedback: LLMNET
version: 26.5.0
title: Cache management
description: Manage the KV cache in long chat sessions with Aspose.LLM for .NET — automatic trimming, manual cleanup, and five strategies for deciding what to keep.
keywords:
- KV cache
- cache cleanup
- CacheCleanupStrategy
- ForceCacheCleanup
- long session
- context overflow
---

Every chat session holds a slice of the KV cache — the per-token state the model uses to condition each next token on prior turns. The cache grows with every message and is bounded by `ContextParameters.ContextSize`. When a new message would overflow, the engine trims the cache according to the session's `CacheCleanupStrategy`.

You can rely on automatic trimming or call `ForceCacheCleanup` at natural boundaries in your application.

## When trimming happens

- **Automatically** — the engine checks the projected context usage before each generation step. When it would exceed `ContextParameters.ContextSize`, the engine trims the cache using the session's `ChatParameters.CacheCleanupStrategy`.
- **Manually** — your code calls `AsposeLLMApi.ForceCacheCleanup(strategy)` at a point that makes sense for your application (topic switch, user-initiated reset, pre-emptive cleanup before a long prompt).

## Strategies

`CacheCleanupStrategy` is an enum in `Aspose.LLM.Abstractions.Models`. Five values, each deciding what to keep.

| Strategy | Keeps | Typical use |
|---|---|---|
| `RemoveOldestMessages` (default) | System prompt + most recent turns | General-purpose. Preserves recency, drops the earliest messages as space is needed. |
| `KeepSystemPromptOnly` | System prompt only | Hard reset. Preserves the role instructions, drops every user and assistant turn. |
| `KeepSystemPromptAndHalf` | System prompt + newer half of history | Balanced. Drops the earlier half, keeps the later half and all new turns. |
| `KeepSystemPromptAndFirstUserMessage` | System prompt + first user turn | Recall-heavy tasks. The original ask stays in context; middle turns drop. |
| `KeepSystemPromptAndLastUserMessage` | System prompt + last user turn | Focused reply. Middle history drops; current question stays. |

### `RemoveOldestMessages` (default)

Walks the history from oldest to newest and evicts messages until there is enough space for the new prompt. The system prompt and the most recent turns survive.

Best for: most scenarios. Preserves local context while allowing arbitrarily long conversations over time.

Drawback: once a turn is evicted, the model can no longer refer back to it.

### `KeepSystemPromptOnly`

Evicts every user and assistant turn from the cache, leaving only the system prompt. The next message starts a fresh dialogue inside the same session.

Best for: explicitly resetting topic mid-session without tearing down the session. Useful when the system prompt encodes important role or format instructions that should survive the reset.

### `KeepSystemPromptAndHalf`

Keeps the system prompt and the second half of the prior history (measured in tokens). Drops the earlier half.

Best for: sessions where gradual eviction is preferable to full clears. Retains roughly half of the conversation's context at the price of less room for fresh turns.

### `KeepSystemPromptAndFirstUserMessage`

Keeps the system prompt and the very first user turn. Everything between is evicted.

Best for: tasks where the original user ask is load-bearing — long debugging sessions, document analyses, iterative refinement on a single input. The model always sees the starting problem statement, even after many follow-ups.

### `KeepSystemPromptAndLastUserMessage`

Keeps the system prompt and the most recent user turn. Middle history drops.

Best for: scenarios focused on the **current** question, where earlier context is no longer relevant. A concise question-answering bot fits this pattern.

## Choosing a strategy

| If the session is… | Prefer |
|---|---|
| A free-flowing conversation | `RemoveOldestMessages` (default) |
| Anchored on a big initial ask, with follow-ups | `KeepSystemPromptAndFirstUserMessage` |
| A sequence of independent questions | `KeepSystemPromptAndLastUserMessage` |
| A topic that you want to reset periodically | `KeepSystemPromptOnly` (force via `ForceCacheCleanup`) |
| Expected to be long, but balanced | `KeepSystemPromptAndHalf` |

## Setting the strategy

On the preset, before creating the API:

```csharp
using Aspose.LLM.Abstractions.Models;

var preset = new Qwen25Preset();
preset.ChatParameters.CacheCleanupStrategy =
    CacheCleanupStrategy.KeepSystemPromptAndFirstUserMessage;

using var api = AsposeLLMApi.Create(preset);
```

The strategy is applied at session creation time. New sessions created from this preset use the chosen strategy for automatic trimming.

## Forcing cleanup manually

```csharp
api.ForceCacheCleanup(CacheCleanupStrategy.KeepSystemPromptOnly);
```

Operates on the **current session**. The strategy argument is independent of the session's default strategy — you can call `ForceCacheCleanup` with any strategy when you want a one-off reset or a stronger trim.

Throws `InvalidOperationException` when no current session exists.

## Practical patterns

### User-initiated reset

```csharp
// User clicks "New topic"
api.ForceCacheCleanup(CacheCleanupStrategy.KeepSystemPromptOnly);
// Next SendMessageAsync starts fresh.
```

### Pre-emptive cleanup before a long prompt

Before sending a 10-page document, clear middle history to maximize room for the new content:

```csharp
api.ForceCacheCleanup(CacheCleanupStrategy.KeepSystemPromptAndFirstUserMessage);
string summary = await api.SendMessageAsync(fullDocument);
```

### Context-aware bot with memory of the original question

```csharp
var preset = new Qwen25Preset();
preset.ChatParameters.SystemPrompt =
    "You are a careful assistant. Always ground your answer in the user's original question.";
preset.ChatParameters.CacheCleanupStrategy =
    CacheCleanupStrategy.KeepSystemPromptAndFirstUserMessage;

using var api = AsposeLLMApi.Create(preset);
```

Even after dozens of follow-ups, the model can still refer back to the first user message.

## What's next

- [Chat sessions](/net/developer-reference/chat-sessions/) — session lifecycle and how cleanup interacts with automatic generation.
- [Chat parameters](/net/developer-reference/parameters/chat/) — set the default strategy per preset.
- [Context parameters](/net/developer-reference/parameters/context/) — `ContextSize` is the ceiling the cleanup policy serves.
- [Multi-turn chat](/net/use-cases/multi-turn-chat/) — cache management in a realistic runnable example.
