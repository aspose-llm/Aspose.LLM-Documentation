---
weight: 20
date: "2026-02-19"
author: "Aleksandr Sudakov"
type: docs
url: /net/developer-reference/chat-sessions/
feedback: LLMNET
title: Chat sessions
description: Starting chat sessions and sending messages with Aspose.LLM for .NET.
keywords:
- chat
- session
- SendMessageAsync
- StartNewChatAsync
---

Aspose.LLM for .NET uses **chat sessions** to maintain conversation context. Each session has an identifier and keeps the dialogue history for that conversation.

## Starting a session

Call `StartNewChatAsync` to create a new chat session:

```csharp
string sessionId = await api.StartNewChatAsync();
// Optional: pass a preset and/or your own session ID
// string sessionId = await api.StartNewChatAsync(preset: myPreset, sessionId: "my-id");
```

The returned `sessionId` identifies the session for subsequent calls.

## Sending a message (auto session)

If you do not need to manage sessions explicitly, use `SendMessageAsync`:

```csharp
string response = await api.SendMessageAsync("Your message here");
```

If no session exists, the API starts one automatically. The response is the model’s reply as a string.

You can pass an optional preset and/or media (e.g. image bytes for vision presets):

```csharp
string response = await api.SendMessageAsync(
    "Describe this image.",
    media: new[] { imageBytes },
    preset: visionPreset,
    cancellationToken: cancellationToken);
```

## Sending a message to a specific session

To send a message to an existing session (e.g. for multi-turn chat):

```csharp
string response = await api.SendMessageToSessionAsync(sessionId, "Follow-up question.");
```

Optional `media` and `cancellationToken` are also supported.

## Current session

The API keeps track of the “current” session when you use `SendMessageAsync` without a session ID. When you call `StartNewChatAsync`, the new session becomes the current one. Use `SendMessageToSessionAsync(sessionId, ...)` to target a specific session regardless of the current one.

## Single instance

Only one `AsposeLLMApi` instance can exist per process. Create it once (e.g. at startup) and reuse it for all sessions and messages until disposal.
