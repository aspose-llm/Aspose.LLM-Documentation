---
weight: 20
date: "2026-02-19"
author: "Aleksandr Sudakov"
type: docs
url: /net/use-cases/multi-turn-chat/
feedback: LLMNET
title: Multi-turn chat
description: Run a multi-turn conversation with an explicit session.
keywords:
- multi-turn
- chat
- session
- StartNewChatAsync
- SendMessageToSessionAsync
---

When you need a sustained conversation and want to control the session (e.g. for multiple users or threads), start a session explicitly and send messages to it by ID.

## Start a session

```csharp
string sessionId = await api.StartNewChatAsync();
```

You can pass an optional preset and/or your own session ID:

```csharp
string sessionId = await api.StartNewChatAsync(preset: myPreset, sessionId: "user-123-conv-1");
```

## Send messages to the session

Use `SendMessageToSessionAsync` with the session ID:

```csharp
string reply1 = await api.SendMessageToSessionAsync(sessionId, "I want to plan a trip.");
string reply2 = await api.SendMessageToSessionAsync(sessionId, "Suggest three cities in Europe.");
string reply3 = await api.SendMessageToSessionAsync(sessionId, "Tell me more about the first one.");
```

The model sees the full conversation history for that session, so it can answer in context.

## Multiple sessions

You can have several sessions (e.g. one per user or conversation) and send messages to any of them:

```csharp
string sessionA = await api.StartNewChatAsync(sessionId: "conv-A");
string sessionB = await api.StartNewChatAsync(sessionId: "conv-B");

await api.SendMessageToSessionAsync(sessionA, "Topic A question.");
await api.SendMessageToSessionAsync(sessionB, "Topic B question.");
```

Remember that only one `AsposeLLMApi` instance is allowed per process; create it once and reuse it for all sessions.
