---
weight: 30
date: "2026-04-23"
author: "Aleksandr Sudakov"
type: docs
url: /net/developer-reference/
feedback: LLMNET
version: 26.5.0
aliases:
- /net/developer-guide/
title: Developer's reference
description: Aspose.LLM for .NET developer reference — presets, chat sessions, session persistence, license, and the API surface.
keywords:
- reference
- preset
- chat
- session
- API
- license
---

This reference describes how to work with Aspose.LLM for .NET once you have the package installed and a license applied. It focuses on concepts, types, and method semantics.

For end-to-end scenarios, see [Use cases](/net/use-cases/). For compact task-oriented snippets, see [Quick wins](/net/quick-wins/).

## Typical flow

1. **Create an API instance** — pick a [preset](/net/developer-reference/presets/) and call `AsposeLLMApi.Create(preset)`.
2. **Start a chat session** — either explicitly with `StartNewChatAsync`, or implicitly by calling `SendMessageAsync`. See [Chat sessions](/net/developer-reference/chat-sessions/).
3. **Send messages** — `SendMessageAsync` for the current session, or `SendMessageToSessionAsync(sessionId, ...)` for a specific one.
4. **Manage cache** — when a session approaches the context limit, call `ForceCacheCleanup(strategy)` to trim the KV cache.
5. **Persist state** — [save and load](/net/developer-reference/session-persistence/) conversations with `SaveChatSession` and `LoadChatSession`.
6. **Dispose** — release native resources with `Dispose` or a `using` block.

## Additional capabilities

- **Default parameters** — `GetDefaultParametersAsync()` returns a tuple of the engine's default inference, context, chat, and sampler parameters.
- **Default preset** — `GetDefaultPreset()` returns a fresh `Qwen25Preset` as a sensible starting point.
- **Single instance** — only one `AsposeLLMApi` instance per process. Disposing the current instance releases the guard.

## Sections

- [Presets](/net/developer-reference/presets/) — preset base class, parameter bags, and override patterns.
- [Chat sessions](/net/developer-reference/chat-sessions/) — starting sessions, sending messages, and cache management.
- [Session persistence](/net/developer-reference/session-persistence/) — saving and loading sessions to disk.
- [License](/net/developer-reference/license/) — the `License` class API.
- [API reference](/net/developer-reference/api-reference/) — link to the full class-level API reference on reference.aspose.com.

## What's next

- [Use cases](/net/use-cases/) — full scenarios with runnable code.
- [Architecture](/net/product-overview/architecture/) — what happens behind the scenes when you call `Create`.
