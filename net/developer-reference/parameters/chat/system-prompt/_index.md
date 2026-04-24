---
weight: 10
date: "2026-04-24"
author: "Alex Golshtein"
type: docs
url: /net/developer-reference/parameters/chat/system-prompt/
feedback: LLMNET
version: 26.5.0
title: SystemPrompt
description: Default system prompt for new chat sessions in Aspose.LLM for .NET — applied at session creation; empty string for presets with no system turn.
keywords:
- SystemPrompt
- system message
- role
- chat instructions
---

`SystemPrompt` is the default system prompt applied to every new session created from this preset. It sets the assistant's role, tone, and constraints.

## Quick reference

| | |
|---|---|
| **Type** | `string` |
| **Default** | `""` (empty) |
| **Category** | Chat session |
| **Field on** | `ChatParameters.SystemPrompt` |

## What it does

When a chat session starts — either explicitly via `StartNewChatAsync` or implicitly on the first `SendMessageAsync` — the engine injects a system turn with this text at the top of the conversation. The model sees the system prompt before any user input and uses it to shape its behavior across the session.

- `""` (default) — no system turn. Some presets (certain Gemma variants) prefer this.
- A short instruction — role and tone. For example, "You are a concise technical assistant."
- A longer instruction — include format constraints, forbidden topics, preferred output structure.

The system prompt is applied once per session. Changes to `SystemPrompt` after `AsposeLLMApi.Create` do not affect already-running sessions.

## When to change it

| Scenario | Value |
|---|---|
| Preset-specific default | Whatever the preset ships with |
| Role specialization | `"You are a ..."` |
| Format enforcement | Explicit format rules |
| Safety / content filtering | Instructions to refuse certain inputs |

Keep system prompts concise — 50-300 tokens. Every token in the system prompt counts against [`ContextParameters.ContextSize`](/net/developer-reference/parameters/context/context-size/).

## Example

```csharp
var preset = new Qwen25Preset();
preset.ChatParameters.SystemPrompt =
    "You are a precise technical assistant. Answer in at most two sentences. " +
    "Say 'I do not know' when you are unsure.";

using var api = AsposeLLMApi.Create(preset);
```

## Interactions

- [`History`](/net/developer-reference/parameters/chat/history/) — seeded history is appended after the system prompt.
- [`CacheCleanupStrategy`](/net/developer-reference/parameters/chat/cache-cleanup-strategy/) — most strategies preserve the system prompt; the cleanup policy anchors on it.
- [`ContextSize`](/net/developer-reference/parameters/context/context-size/) — system prompt consumes tokens from the window.

## What's next

- [System prompt recipes](/net/use-cases/system-prompt-recipes/) — effective patterns.
- [CacheCleanupStrategy](/net/developer-reference/parameters/chat/cache-cleanup-strategy/) — how the system prompt interacts with cache trimming.
- [Chat parameters hub](/net/developer-reference/parameters/chat/) — all chat knobs.
