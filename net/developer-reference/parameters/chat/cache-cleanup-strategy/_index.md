---
weight: 40
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/chat/cache-cleanup-strategy/
feedback: LLMNET
version: 26.5.0
title: CacheCleanupStrategy
description: Policy for KV cache trimming when context fills up in Aspose.LLM for .NET — five strategies ranging from full reset to preserving the first user turn.
keywords:
- CacheCleanupStrategy
- KV cache
- eviction
- policy
- conversation trimming
---

`CacheCleanupStrategy` is the policy the engine applies when a session's KV cache would overflow [`ContextSize`](/net/developer-reference/parameters/context/context-size/). Five named strategies, each keeping a different subset of history.

## Quick reference

| | |
|---|---|
| **Type** | `CacheCleanupStrategy` enum |
| **Default** | `RemoveOldestMessages` |
| **Values** | 5 — see table below |
| **Category** | Chat session |
| **Field on** | `ChatParameters.CacheCleanupStrategy` |

## What it does

When the next generation step would exceed the context window, the engine trims the cache per the active strategy, freeing tokens before continuing. The policy is applied automatically during generation and can also be triggered explicitly via `AsposeLLMApi.ForceCacheCleanup(strategy)`.

| Strategy | Keeps | Typical use |
|---|---|---|
| `RemoveOldestMessages` (default) | System prompt + most recent turns | General-purpose; preserves recency. |
| `KeepSystemPromptOnly` | System prompt only | Hard reset of session context. |
| `KeepSystemPromptAndHalf` | System prompt + newer half of history | Balanced recall and room for new turns. |
| `KeepSystemPromptAndFirstUserMessage` | System prompt + first user turn | Recall-heavy tasks where the original ask matters. |
| `KeepSystemPromptAndLastUserMessage` | System prompt + most recent user turn | Focus on current question, drop middle. |

## When to change it

| Scenario | Value |
|---|---|
| Default conversational chat | `RemoveOldestMessages` |
| Anchored on a big original ask (debugging, iterative refinement) | `KeepSystemPromptAndFirstUserMessage` |
| Sequence of independent Q&A | `KeepSystemPromptAndLastUserMessage` |
| Hard reset when switching topics | `KeepSystemPromptOnly` via `ForceCacheCleanup` |
| Long dialogues with gradual trimming | `KeepSystemPromptAndHalf` |

## Example

```csharp
using Aspose.LLM.Abstractions.Models;

var preset = new Qwen25Preset();
preset.ChatParameters.SystemPrompt =
    "You are a careful analyst. Always ground your answer in the user's original ask.";
preset.ChatParameters.CacheCleanupStrategy =
    CacheCleanupStrategy.KeepSystemPromptAndFirstUserMessage;

using var api = AsposeLLMApi.Create(preset);
// Even after 50 follow-ups, the model can still refer back to the first user turn.
```

Force a reset mid-session:

```csharp
api.ForceCacheCleanup(CacheCleanupStrategy.KeepSystemPromptOnly);
```

## Interactions

- [`SystemPrompt`](/net/developer-reference/parameters/chat/system-prompt/) — all strategies preserve it.
- [`ContextSize`](/net/developer-reference/parameters/context/context-size/) — the ceiling this strategy serves.
- [`DefragThreshold`](/net/developer-reference/parameters/context/defrag-threshold/) — compacts holes left behind by cleanup.
- `AsposeLLMApi.ForceCacheCleanup(strategy)` — manual trigger with an override strategy.

## What's next

- [Cache management](/net/developer-reference/cache-management/) — full guide with practical patterns.
- [Multi-turn chat use case](/net/use-cases/multi-turn-chat/) — cache management in practice.
- [Chat parameters hub](/net/developer-reference/parameters/chat/) — all chat knobs.
