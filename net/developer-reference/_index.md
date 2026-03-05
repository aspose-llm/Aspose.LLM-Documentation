---
weight: 30
date: "2026-02-19"
author: "Aleksandr Sudakov"
type: docs
url: /net/developer-reference/
feedback: LLMNET
aliases:
- /net/developer-guide/
title: Developer's reference
description: Aspose.LLM for .NET developer reference: presets, chat sessions, persistence, license.
keywords:
- reference
- preset
- chat
- session
- API
---

This reference describes how to work with Aspose.LLM for .NET.

Typical flow:

1. **Create an API instance** — Choose a [preset](/net/developer-reference/presets/) and call `AsposeLLMApi.Create(preset)`.
2. **Chat sessions** — [Start a session](/net/developer-reference/chat-sessions/) with `StartNewChatAsync`, then send messages with `SendMessageAsync` or `SendMessageToSessionAsync`.
3. **Session persistence** — [Save and load](/net/developer-reference/session-persistence/) conversation state with `SaveChatSession` and `LoadChatSession`.
4. **License** — Apply a [license](/net/developer-reference/license/) before using the API.

Additional capabilities:

- **Default parameters** — `GetDefaultParametersAsync()` and `GetDefaultPreset()` return engine defaults.
- **Cache cleanup** — `ForceCacheCleanup(strategy)` to free context cache (e.g. when switching conversations).
- **Single instance** — Only one `AsposeLLMApi` instance per process.

## Sections

- [Presets](/net/developer-reference/presets/) — Preset types and usage.
- [Chat sessions](/net/developer-reference/chat-sessions/) — Starting sessions and sending messages.
- [Session persistence](/net/developer-reference/session-persistence/) — Saving and loading chat sessions.
- [License](/net/developer-reference/license/) — Applying and checking license.
- [API reference](https://reference.aspose.com/net/) — Full API documentation (external).
