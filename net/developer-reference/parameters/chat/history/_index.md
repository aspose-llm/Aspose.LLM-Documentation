---
weight: 20
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/chat/history/
feedback: LLMNET
version: 26.5.0
title: History
description: Pre-seeded conversation history for new chat sessions in Aspose.LLM for .NET — inject user/assistant turns for few-shot priming.
keywords:
- History
- chat history
- few-shot
- priming
- ChatMessage
---

`History` is an optional list of `ChatMessage` objects used to pre-seed every new session. Useful for few-shot priming, restoring a conversation from external storage, or warming the model with a specific example set.

## Quick reference

| | |
|---|---|
| **Type** | `List<ChatMessage>?` |
| **Default** | `null` (no pre-seeded history) |
| **Category** | Chat session |
| **Field on** | `ChatParameters.History` |

## What it does

When a new session is created, the engine appends each entry of `History` after the system prompt, before any user message in the current turn. The model sees these turns as if they had been exchanged earlier.

- `null` (default) — fresh session with only the system prompt.
- Explicit list — every new session starts with these turns already in the KV cache.

`History` is applied at session creation; changing the list after `Create` has no effect on already-running sessions.

## When to change it

| Scenario | Value |
|---|---|
| Default — blank session | `null` |
| Few-shot priming for consistent output | 2-4 example turns |
| Long-running personality — reinforce tone with examples | 3-5 stylistic turns |
| Seed from stored transcript | Application-specific |

## Example

```csharp
using Aspose.LLM.Abstractions.Models;

var preset = new Qwen25Preset();
preset.ChatParameters.History = new List<ChatMessage>
{
    ChatMessage.CreateUserMessage("Translate to French: The cat sleeps."),
    ChatMessage.CreateAssistantMessage("Le chat dort."),
    ChatMessage.CreateUserMessage("Translate to French: The dog barks."),
    ChatMessage.CreateAssistantMessage("Le chien aboie."),
};

using var api = AsposeLLMApi.Create(preset);
// Every new session now starts with these four priming turns.
```

## Interactions

- [`SystemPrompt`](/net/developer-reference/parameters/chat/system-prompt/) — applied before `History`.
- [`ContextSize`](/net/developer-reference/parameters/context/context-size/) — pre-seeded turns consume tokens from the window.
- `ChatMessage.CreateUserMessage` / `CreateAssistantMessage` / `CreateSystemMessage` — factories for building entries.

## What's next

- [Chat history reference](/net/developer-reference/chat-sessions/chat-history/) — `ChatMessage` structure in detail.
- [System prompt recipes](/net/use-cases/system-prompt-recipes/) — priming patterns.
- [Chat parameters hub](/net/developer-reference/parameters/chat/) — all chat knobs.
